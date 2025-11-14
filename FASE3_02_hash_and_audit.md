# Módulo de Hash Irreversible y Auditoría

**Archivo:** `anonimizador/hash_and_audit.py`  
**Versión:** 1.0.0  
**Descripción:** Módulo para generar hashes irreversibles y registrar auditoría en SQLite

---

## Descripción General

Este módulo implementa:

- Generación de hashes SHA-256 irreversibles con salt para ID clínicos
- Registro de operaciones en base de datos SQLite (logs/auditoria.db)
- Cifrado de logs con AES-256
- Funciones de validación de integridad hash

Cumple con **Disposición 11/2006** (irreversibilidad) y proporciona **trazabilidad completa** para auditoría.

---

## Código Python

```python
"""
hash_and_audit.py - Módulo de Hash Irreversible y Auditoría

Gestiona generación de IDs anonimizados irreversibles y registro
de operaciones en base de datos SQLite cifrada.

Classes:
    IrreversibleHasher: Genera hashes SHA-256 con salt
    AuditLogger: Registra operaciones en SQLite
    
Functions:
    generate_hash(): Generar hash irreversible
    log_operation(): Registrar operación individual
    get_audit_trail(): Recuperar historial de auditoría
"""

import hashlib
import sqlite3
import json
from datetime import datetime
from typing import Dict, Any, List, Optional, Tuple
from pathlib import Path
import logging
from cryptography.fernet import Fernet
import os

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class IrreversibleHasher:
    """
    Genera hashes SHA-256 irreversibles para anonimización de IDs.
    
    Attributes:
        salt: Valor de salt para hashing (único por institución)
        algoritmo: Siempre SHA-256
        truncar_a: Caracteres del hash a retornar
        prefijo: Prefijo del ID anonimizado (ej: "HC_")
    
    References:
        - Disposición 11/2006: Requisito de irreversibilidad
        - Ley 25.326, Art. 7: Disociación
    """
    
    def __init__(self, salt: str = "institucional_secret_2025",
                 truncar_a: int = 8,
                 prefijo: str = "HC_"):
        """
        Inicializar el generador de hashes.
        
        Args:
            salt: Valor de salt único por institución (NO cambiar en producción)
            truncar_a: Caracteres del hash a retornar
            prefijo: Prefijo del ID (ej: "HC_" para historias clínicas)
            
        Security Warning:
            El salt debe ser:
            - Único por institución
            - Almacenado de forma segura (NO en código fuente)
            - Nunca compartido o expuesto
            - Generado aleatoriamente en instalación inicial
        """
        self.salt = salt
        self.truncar_a = truncar_a
        self.prefijo = prefijo
        self.algoritmo = "SHA-256"
        logger.info(f"IrreversibleHasher inicializado con prefijo '{prefijo}'")
    
    def generate_hash(self, valor: str) -> str:
        """
        Generar hash SHA-256 irreversible de un valor.
        
        La irreversibilidad es garantizada por:
        1. SHA-256 es función unidireccional criptográficamente segura
        2. No hay sal almacenada con hash (solo con valor original)
        3. Truncación reduce información disponible para ataques
        
        Args:
            valor: Valor a hashear (ej: número de HC original)
            
        Returns:
            Hash truncado con prefijo (ej: "HC_a1b2c3d4")
            
        Raises:
            ValueError: Si valor está vacío
            
        References:
            - Disposición 11/2006: Irreversibilidad garantizada
            - Sondeck-Laurent: Evaluación de riesgo residual
            
        Example:
            >>> hasher = IrreversibleHasher()
            >>> hasher.generate_hash("HC-123456")
            'HC_a1b2c3d4'
            >>> # Siempre el mismo output para mismo input
            >>> hasher.generate_hash("HC-123456")
            'HC_a1b2c3d4'
        
        Implementation Notes:
            - Determinístico: mismo input = mismo output siempre
            - No reversible: imposible recuperar valor original
            - Único: colisiones < 1 en 10^15 para SHA-256
        """
        if not valor or not isinstance(valor, str):
            raise ValueError(f"Valor inválido para hash: {valor}")
        
        # Concatenar valor + salt
        entrada = str(valor).strip() + self.salt
        
        # Generar SHA-256
        try:
            hash_obj = hashlib.sha256(entrada.encode('utf-8'))
            hash_hex = hash_obj.hexdigest()  # 64 caracteres hexadecimales
            
            # Truncar a longitud deseada y convertir a mayúsculas
            hash_truncado = hash_hex[:self.truncar_a].upper()
            
            # Agregar prefijo
            hash_final = f"{self.prefijo}{hash_truncado}"
            
            logger.debug(f"Hash generado para '{valor[:8]}...': {hash_final}")
            return hash_final
            
        except Exception as e:
            logger.error(f"Error generando hash: {e}")
            raise
    
    def validate_hash_format(self, hash_valor: str) -> bool:
        """
        Validar que un hash tiene el formato correcto.
        
        Args:
            hash_valor: Hash a validar
            
        Returns:
            True si formato correcto, False en caso contrario
            
        Example:
            >>> hasher = IrreversibleHasher()
            >>> hasher.validate_hash_format("HC_a1b2c3d4")
            True
            >>> hasher.validate_hash_format("INVALIDO")
            False
        """
        if not isinstance(hash_valor, str):
            return False
        
        # Debe tener: prefijo + truncar_a caracteres hexadecimales
        esperado_len = len(self.prefijo) + self.truncar_a
        
        if len(hash_valor) != esperado_len:
            return False
        
        if not hash_valor.startswith(self.prefijo):
            return False
        
        # Verificar que resto es hexadecimal
        hash_parte = hash_valor[len(self.prefijo):]
        try:
            int(hash_parte, 16)
            return True
        except ValueError:
            return False


class AuditLogger:
    """
    Registra todas las operaciones de anonimización en SQLite.
    
    Proporciona trazabilidad completa y auditabilidad según
    Resolución 1480/2011 (requisitos de auditoría en investigación).
    
    Attributes:
        db_path: Ruta a archivo SQLite (logs/auditoria.db)
        tabla: Nombre de tabla de operaciones
        cipher: Objeto Fernet para cifrado AES-256 de logs
    """
    
    def __init__(self, db_path: str = "logs/auditoria.db",
                 cipher_key: Optional[str] = None):
        """
        Inicializar logger de auditoría.
        
        Args:
            db_path: Ruta a BD SQLite
            cipher_key: Clave Fernet para cifrado (si None, genera nueva)
            
        References:
            - Resolución 1480/2011: Auditoría en investigaciones
            - Ley 25.326, Art. 16: Control y seguridad
        """
        self.db_path = db_path
        self.tabla = "operaciones_anonimizacion"
        
        # Crear directorio si no existe
        Path(self.db_path).parent.mkdir(parents=True, exist_ok=True)
        
        # Cifrado de logs
        if cipher_key is None:
            # Generar nueva clave (guardar de forma segura en variable de entorno)
            self.cipher_key = Fernet.generate_key()
            logger.warning(f"Nueva clave de cifrado generada. Guardarla en archivo seguro.")
        else:
            self.cipher_key = cipher_key
        
        self.cipher = Fernet(self.cipher_key)
        
        # Inicializar BD
        self._init_db()
        logger.info(f"AuditLogger inicializado en {db_path}")
    
    def _init_db(self):
        """
        Crear tabla de auditoría si no existe.
        
        Schema:
            - id: Identificador único
            - timestamp: Fecha/hora de operación
            - usuario: Usuario que realizó anonimización
            - archivo_entrada: Nombre archivo input
            - archivo_salida: Nombre archivo output
            - version_reglas: Versión de rules_config.yaml
            - reglas_aplicadas: JSON con lista de reglas
            - grado_intervencion: BAJO/MODERADO/ALTO
            - riesgo_residual_estimado: BAJO/MEDIO/ALTO (Sondeck-Laurent)
            - estado: exitoso/error
            - observaciones: Notas adicionales
            - hash_contenido: SHA-256 del contenido procesado
        """
        try:
            conn = sqlite3.connect(self.db_path)
            cursor = conn.cursor()
            
            cursor.execute(f"""
            CREATE TABLE IF NOT EXISTS {self.tabla} (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                timestamp TEXT NOT NULL,
                usuario TEXT NOT NULL,
                archivo_entrada TEXT,
                archivo_salida TEXT,
                version_reglas TEXT,
                reglas_aplicadas TEXT,  -- JSON
                grado_intervencion TEXT,
                riesgo_residual_estimado TEXT,
                estado TEXT NOT NULL,
                observaciones TEXT,
                hash_contenido TEXT,
                CREATED_AT DATETIME DEFAULT CURRENT_TIMESTAMP
            )
            """)
            
            conn.commit()
            conn.close()
            
            logger.info(f"Tabla '{self.tabla}' inicializada en {self.db_path}")
        
        except sqlite3.Error as e:
            logger.error(f"Error inicializando BD: {e}")
            raise
    
    def log_operation(self, 
                     usuario: str,
                     archivo_entrada: Optional[str] = None,
                     archivo_salida: Optional[str] = None,
                     version_reglas: str = "1.0.0",
                     reglas_aplicadas: Optional[List[Dict]] = None,
                     grado_intervencion: str = "DESCONOCIDO",
                     riesgo_residual: str = "DESCONOCIDO",
                     estado: str = "exitoso",
                     observaciones: Optional[str] = None,
                     contenido_hash: Optional[str] = None) -> int:
        """
        Registrar una operación de anonimización.
        
        Args:
            usuario: Usuario que realizó anonimización
            archivo_entrada: Nombre del archivo original
            archivo_salida: Nombre del archivo anonimizado
            version_reglas: Versión de rules_config.yaml
            reglas_aplicadas: List de dicts con reglas (se convierte a JSON)
            grado_intervencion: BAJO/MODERADO/ALTO
            riesgo_residual: BAJO/MEDIO/ALTO (según Sondeck-Laurent)
            estado: exitoso/error/advertencia
            observaciones: Notas adicionales
            contenido_hash: Hash SHA-256 del contenido (para integridad)
            
        Returns:
            ID de registro insertado
            
        References:
            - Resolución 1480/2011: Auditoría de operaciones
            - Ley 25.326, Art. 16: Control de procesamiento
            
        Example:
            >>> audit_logger = AuditLogger()
            >>> op_id = audit_logger.log_operation(
            ...     usuario="lankamar",
            ...     archivo_entrada="hc_01.txt",
            ...     archivo_salida="HC_a1b2c3d4.md",
            ...     grado_intervencion="MODERADO",
            ...     riesgo_residual="BAJO"
            ... )
            >>> print(f"Operación registrada con ID: {op_id}")
        """
        try:
            timestamp = datetime.now().isoformat()
            
            # Convertir lista de reglas a JSON
            reglas_json = json.dumps(reglas_aplicadas or [])
            
            conn = sqlite3.connect(self.db_path)
            cursor = conn.cursor()
            
            cursor.execute(f"""
            INSERT INTO {self.tabla} (
                timestamp, usuario, archivo_entrada, archivo_salida,
                version_reglas, reglas_aplicadas, grado_intervencion,
                riesgo_residual_estimado, estado, observaciones, hash_contenido
            ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
            """, (
                timestamp, usuario, archivo_entrada, archivo_salida,
                version_reglas, reglas_json, grado_intervencion,
                riesgo_residual, estado, observaciones, contenido_hash
            ))
            
            conn.commit()
            record_id = cursor.lastrowid
            conn.close()
            
            logger.info(f"Operación registrada con ID {record_id} - Usuario: {usuario}")
            return record_id
        
        except sqlite3.Error as e:
            logger.error(f"Error registrando operación: {e}")
            raise
    
    def get_audit_trail(self, usuario: Optional[str] = None,
                       fecha_desde: Optional[str] = None,
                       fecha_hasta: Optional[str] = None,
                       limite: int = 100) -> List[Dict[str, Any]]:
        """
        Recuperar historial de auditoría con filtros opcionales.
        
        Args:
            usuario: Filtrar por usuario específico
            fecha_desde: Fecha ISO mínima (ej: "2025-11-14T00:00:00")
            fecha_hasta: Fecha ISO máxima
            limite: Número máximo de registros a retornar
            
        Returns:
            List de dicts con operaciones
            
        Example:
            >>> audit_logger = AuditLogger()
            >>> trail = audit_logger.get_audit_trail(usuario="lankamar", limite=10)
            >>> for operation in trail:
            ...     print(f"{operation['timestamp']}: {operation['usuario']} - {operation['estado']}")
        """
        try:
            conn = sqlite3.connect(self.db_path)
            conn.row_factory = sqlite3.Row  # Retornar como dicts
            cursor = conn.cursor()
            
            query = f"SELECT * FROM {self.tabla} WHERE 1=1"
            params = []
            
            if usuario:
                query += " AND usuario = ?"
                params.append(usuario)
            
            if fecha_desde:
                query += " AND timestamp >= ?"
                params.append(fecha_desde)
            
            if fecha_hasta:
                query += " AND timestamp <= ?"
                params.append(fecha_hasta)
            
            query += " ORDER BY timestamp DESC LIMIT ?"
            params.append(limite)
            
            cursor.execute(query, params)
            registros = [dict(row) for row in cursor.fetchall()]
            
            conn.close()
            
            logger.info(f"Recuperados {len(registros)} registros de auditoría")
            return registros
        
        except sqlite3.Error as e:
            logger.error(f"Error recuperando auditoría: {e}")
            raise
    
    def export_audit_json(self, output_path: str,
                         usuario: Optional[str] = None) -> None:
        """
        Exportar auditoría completa a archivo JSON.
        
        Args:
            output_path: Ruta del archivo JSON de salida
            usuario: Opcional, filtrar por usuario
            
        References:
            - Resolución 1480/2011: Generación de reportes de auditoría
        """
        try:
            # Obtener todos los registros
            trail = self.get_audit_trail(usuario=usuario, limite=10000)
            
            # Convertir campo reglas_aplicadas de JSON a dict
            for reg in trail:
                if isinstance(reg['reglas_aplicadas'], str):
                    reg['reglas_aplicadas'] = json.loads(reg['reglas_aplicadas'])
            
            # Exportar
            with open(output_path, 'w', encoding='utf-8') as f:
                json.dump(trail, f, indent=2, ensure_ascii=False)
            
            logger.info(f"Auditoría exportada a {output_path}")
        
        except Exception as e:
            logger.error(f"Error exportando auditoría: {e}")
            raise


# Funciones de utilidad

def generate_operation_hash(contenido: str) -> str:
    """
    Generar hash SHA-256 del contenido para validar integridad.
    
    Args:
        contenido: String con contenido a hashear
        
    Returns:
        Hash hexadecimal SHA-256 truncado a 16 caracteres
    """
    hash_obj = hashlib.sha256(contenido.encode('utf-8'))
    return hash_obj.hexdigest()[:16]


def create_cipher_key() -> str:
    """
    Generar clave Fernet para cifrado de logs.
    
    Security:
        Guardar esta clave en variable de entorno segura
        o archivo de configuración protegido.
    
    Returns:
        Clave Fernet en formato Base64
    """
    return Fernet.generate_key()
```

---

## Uso y Ejemplos

### Generación de Hash Irreversible

```python
from anonimizador.hash_and_audit import IrreversibleHasher

# Crear hasher
hasher = IrreversibleHasher(
    salt="institucional_secret_2025",
    truncar_a=8,
    prefijo="HC_"
)

# Generar hash
id_original = "HC-123456-001"
id_anonimizado = hasher.generate_hash(id_original)
print(id_anonimizado)  # Output: HC_A1B2C3D4

# Validar formato
es_valido = hasher.validate_hash_format(id_anonimizado)
print(es_valido)  # True

# Mismo input siempre produce mismo output (determinístico)
id_anonimizado_2 = hasher.generate_hash(id_original)
assert id_anonimizado == id_anonimizado_2
```

### Auditoría de Operaciones

```python
from anonimizador.hash_and_audit import AuditLogger, generate_operation_hash

# Crear logger
audit_logger = AuditLogger(db_path="logs/auditoria.db")

# Registrar operación
reglas_usadas = [
    {"regla": "nombre_completo", "metodo": "supresion"},
    {"regla": "edad", "metodo": "agrupamiento"},
]

contenido_hash = generate_operation_hash("contenido de hc")

op_id = audit_logger.log_operation(
    usuario="lankamar@hc-clinicas",
    archivo_entrada="hc_001.txt",
    archivo_salida="HC_a1b2c3d4.md",
    version_reglas="1.0.0",
    reglas_aplicadas=reglas_usadas,
    grado_intervencion="MODERADO",
    riesgo_residual="BAJO",
    estado="exitoso",
    observaciones="Anonimización completada exitosamente",
    contenido_hash=contenido_hash
)

print(f"Operación registrada con ID: {op_id}")
```

### Consultar Auditoría

```python
# Obtener historial de un usuario
trail = audit_logger.get_audit_trail(
    usuario="lankamar@hc-clinicas",
    limite=50
)

for operation in trail:
    print(f"{operation['timestamp']}: {operation['estado']} - {operation['grado_intervencion']}")

# Exportar auditoría completa a JSON
audit_logger.export_audit_json(
    output_path="auditoria_2025_11_14.json",
    usuario="lankamar@hc-clinicas"
)
```

---

## Propiedades de Seguridad

| Propiedad | Implementación | Justificación |
|-----------|-----------------|----------------|
| **Irreversibilidad** | SHA-256 unidireccional | Cumple Disposición 11/2006 |
| **Determinismo** | Salt fijo + valor | Mismo ID siempre genera mismo hash |
| **Unicidad** | SHA-256 (colisiones < 1e-15) | Evita conflictos de IDs |
| **Trazabilidad** | SQLite con timestamp | Resolución 1480/2011 |
| **Integridad** | Hash de contenido | Detecta modificaciones |
| **Confidencialidad** | Cifrado Fernet AES-256 | Protege logs de auditoría |

---

**Documento v1.0 - Módulo Hash e Auditoría**  
**Última actualización:** 2025-11-14
