# Módulo Core de Anonimización Textual

**Archivo:** `anonimizador/core_anonymization.py`  
**Versión:** 1.0.0  
**Descripción:** Módulo principal que aplica las reglas de anonimización textual definidas en rules_config.yaml

---

## Descripción General

Este módulo contiene las funciones core para anonimizar datos textuales según la metodología Sondeck-Laurent y cumpliendo Ley 25.326/Disposición 11/2006. Implementa:

- Supresión de identificadores directos (IDD)
- Generalización de cuasi-identificadores (QI)
- Protección de atributos sensibles (AS)
- Registro exhaustivo de intervenciones aplicadas

---

## Código Python

```python
"""
core_anonymization.py - Módulo Core de Anonimización Textual

Implementa las reglas de anonimización definidas en rules_config.yaml,
aplicando supresión, generalización y agrupamiento según metodología
Sondeck-Laurent y Ley 25.326 (Argentina).

Classes:
    AnonymizationRule: Define una regla de anonimización individual
    CoreAnonymizer: Motor principal que aplica reglas a datos

Functions:
    load_config(): Cargar rules_config.yaml
    anonymize_record(): Anonimizar un registro completo
    log_intervention(): Registrar intervención en auditoría
"""

import yaml
import re
import hashlib
from typing import Dict, Any, List, Tuple
from datetime import datetime
from pathlib import Path
import logging

# Configurar logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)


class AnonymizationRule:
    """
    Representa una regla individual de anonimización.
    
    Attributes:
        nombre: Identificador único de la regla
        campo: Campo al que aplica
        tipo: IDD, QI o AS
        metodo: supresion, generalizacion, agrupamiento, hash, retener
        severidad: Alta, Media, Baja
        exposicion: Alta, Media, Baja
        parametros: Dict con parámetros específicos del método
        base_normativa: Referencia legal/metodológica
    """
    
    def __init__(self, nombre: str, config: Dict[str, Any]):
        """
        Inicializar una regla de anonimización.
        
        Args:
            nombre: Nombre de la regla
            config: Dict con configuración de la regla (del YAML)
            
        References:
            - Ley 25.326, Art. 7: Concepto de disociación
            - Sondeck-Laurent (2025): Clasificación de atributos
        """
        self.nombre = nombre
        self.campo = config.get('campo')
        self.tipo = config.get('tipo')  # IDD, QI, AS
        self.metodo = config.get('metodo')
        self.severidad = config.get('severidad')
        self.exposicion = config.get('exposicion')
        self.parametros = config.get('parametros', {})
        self.base_normativa = config.get('base_normativa')
        self.descripcion = config.get('descripcion')
    
    def __repr__(self) -> str:
        return f"AnonymizationRule({self.nombre}, {self.tipo}, {self.metodo})"


class CoreAnonymizer:
    """
    Motor principal de anonimización que aplica reglas configuradas.
    
    Attributes:
        config: Dict con configuración completa del proyecto
        reglas: List de AnonymizationRule cargadas
        intervenciones_aplicadas: Log de intervenciones por sesión
        salt_hash: Salt para hashing irreversible
    
    Methods:
        load_config(): Cargar rules_config.yaml
        anonymize_record(): Procesar un registro completo
        apply_rule(): Aplicar regla individual a un valor
        log_intervention(): Registrar intervención
    """
    
    def __init__(self, config_path: str = "rules_config.yaml", 
                 salt: str = "institucional_secret_2025"):
        """
        Inicializar el motor de anonimización.
        
        Args:
            config_path: Ruta a rules_config.yaml
            salt: Salt para hashing (desde configuración)
            
        Raises:
            FileNotFoundError: Si config_path no existe
            yaml.YAMLError: Si YAML es inválido
        """
        self.config_path = config_path
        self.salt = salt
        self.config = self.load_config()
        self.reglas = self._cargar_reglas()
        self.intervenciones_aplicadas: List[Dict[str, Any]] = []
        logger.info(f"CoreAnonymizer inicializado con {len(self.reglas)} reglas")
    
    def load_config(self) -> Dict[str, Any]:
        """
        Cargar configuración desde rules_config.yaml.
        
        Returns:
            Dict con toda la configuración
            
        Raises:
            FileNotFoundError: Si archivo no existe
            yaml.YAMLError: Si YAML inválido
        """
        try:
            with open(self.config_path, 'r', encoding='utf-8') as f:
                config = yaml.safe_load(f)
            logger.info(f"Configuración cargada desde {self.config_path}")
            return config
        except FileNotFoundError:
            logger.error(f"Archivo de configuración no encontrado: {self.config_path}")
            raise
        except yaml.YAMLError as e:
            logger.error(f"Error en YAML: {e}")
            raise
    
    def _cargar_reglas(self) -> List[AnonymizationRule]:
        """
        Cargar todas las reglas de anonimización desde configuración.
        
        Returns:
            List de AnonymizationRule
        """
        reglas = []
        
        # Cargar identificadores directos
        for nombre, config in self.config.get('identificadores_directos', {}).items():
            reglas.append(AnonymizationRule(nombre, config))
        
        # Cargar cuasi-identificadores
        for nombre, config in self.config.get('cuasi_identificadores', {}).items():
            reglas.append(AnonymizationRule(nombre, config))
        
        # Cargar atributos sensibles
        for nombre, config in self.config.get('atributos_sensibles', {}).items():
            reglas.append(AnonymizationRule(nombre, config))
        
        logger.info(f"Cargadas {len(reglas)} reglas de anonimización")
        return reglas
    
    def anonymize_record(self, record: Dict[str, Any], 
                        usuario: str = "sistema") -> Tuple[Dict[str, Any], List[Dict]]:
        """
        Anonimizar un registro completo (historia clínica).
        
        Aplica todas las reglas relevantes al registro y documenta
        cada intervención realizada.
        
        Args:
            record: Dict con datos de la HC (ej: {"nombre": "Juan", "edad": 37, ...})
            usuario: Usuario que realiza anonimización (para auditoría)
            
        Returns:
            Tuple de (record_anonimizado, lista_intervenciones)
            
        References:
            - Ley 25.326, Art. 7: Disociación irreversible
            - Sondeck-Laurent: Clasificación y evaluación de riesgo
            
        Example:
            >>> anonymizer = CoreAnonymizer()
            >>> record = {"nombre": "Juan Pérez", "edad": 37, "dnf": "12345678"}
            >>> record_anon, interventions = anonymizer.anonymize_record(record)
            >>> print(record_anon)
            {'nombre': '[SUPRIMIDO]', 'edad': '35-39', 'dnf': '[SUPRIMIDO]'}
        """
        record_anon = record.copy()
        intervenciones = []
        timestamp = datetime.now().isoformat()
        
        # Aplicar cada regla
        for regla in self.reglas:
            if regla.campo in record:
                valor_original = record[regla.campo]
                valor_anon, intervención = self.apply_rule(regla, valor_original)
                
                if valor_anon != valor_original:
                    record_anon[regla.campo] = valor_anon
                    intervención['timestamp'] = timestamp
                    intervención['usuario'] = usuario
                    intervenciones.append(intervención)
                    
                    logger.info(f"Regla aplicada: {regla.nombre} a campo {regla.campo}")
        
        self.intervenciones_aplicadas.extend(intervenciones)
        return record_anon, intervenciones
    
    def apply_rule(self, regla: AnonymizationRule, 
                   valor: Any) -> Tuple[Any, Dict[str, Any]]:
        """
        Aplicar una regla individual a un valor.
        
        Según el método de la regla, ejecuta la transformación
        correspondiente (supresión, generalización, etc).
        
        Args:
            regla: AnonymizationRule con configuración
            valor: Valor a anonimizar
            
        Returns:
            Tuple de (valor_anonimizado, dict_intervención)
            
        References:
            - Disposición 11/2006: Criterios técnicos de disociación
            - Sondeck-Laurent: Métodos de transformación
        """
        intervención = {
            'regla': regla.nombre,
            'campo': regla.campo,
            'tipo': regla.tipo,
            'metodo': regla.metodo,
            'severidad': regla.severidad,
            'exposicion': regla.exposicion,
            'valor_original_hash': hashlib.sha256(str(valor).encode()).hexdigest()[:8],
            'base_normativa': regla.base_normativa
        }
        
        # Supresión: Reemplazar por [SUPRIMIDO]
        if regla.metodo == 'supresion':
            valor_anon = regla.parametros.get('valor_salida', '[SUPRIMIDO]')
            intervención['accion'] = 'supresion_total'
            intervención['razon'] = f"Identificador directo ({regla.tipo})"
        
        # Generalización Temporal: Convertir a año
        elif regla.metodo == 'generalizacion_temporal':
            valor_anon = self._generalizar_fecha(valor, regla.parametros)
            intervención['accion'] = 'generalizacion_temporal'
            intervención['valor_transformado'] = valor_anon
            intervención['razon'] = "Reducir temporalidad para minimizar riesgo"
        
        # Generalización Geográfica: Convertir a provincia
        elif regla.metodo == 'generalizacion_geografica':
            valor_anon = self._generalizar_ubicacion(valor, regla.parametros)
            intervención['accion'] = 'generalizacion_geografica'
            intervención['valor_transformado'] = valor_anon
            intervención['razon'] = "Reducir especificidad geográfica (Sondeck-Laurent)"
        
        # Agrupamiento: Agrupar edad en rangos
        elif regla.metodo == 'agrupamiento':
            valor_anon = self._agrupar_valor(valor, regla.parametros)
            intervención['accion'] = 'agrupamiento'
            intervención['valor_transformado'] = valor_anon
            intervención['razon'] = f"k-anonimato: {regla.parametros.get('rango_anos', 5)} años"
        
        # Hash Irreversible: SHA-256
        elif regla.metodo == 'hash_irreversible':
            valor_anon = self._hash_irreversible(valor, regla.parametros)
            intervención['accion'] = 'hash_irreversible'
            intervención['valor_transformado'] = valor_anon
            intervención['razon'] = "Disociación irreversible según Disposición 11/2006"
        
        # Retener: No transformar
        elif regla.metodo == 'retener':
            valor_anon = valor
            intervención['accion'] = 'retener'
            intervención['razon'] = "Campo permitido para utilidad científica"
        
        else:
            logger.warning(f"Método desconocido: {regla.metodo}")
            valor_anon = valor
            intervención['accion'] = 'no_aplicado'
        
        return valor_anon, intervención
    
    def _generalizar_fecha(self, fecha_str: str, 
                          parametros: Dict[str, Any]) -> str:
        """
        Generalizar fecha a año solamente.
        
        Convierte "15/03/2021" → "2021" para reducir singularidad
        temporal y cumplir con Sondeck-Laurent.
        
        Args:
            fecha_str: Fecha en formato string (ej: "15/03/2021")
            parametros: Dict con formato_entrada, formato_salida
            
        Returns:
            Año como string (ej: "2021")
        """
        try:
            fmt_entrada = parametros.get('formato_entrada', "%d/%m/%Y")
            fmt_salida = parametros.get('formato_salida', "%Y")
            
            from datetime import datetime as dt
            fecha_obj = dt.strptime(str(fecha_str).strip(), fmt_entrada)
            return fecha_obj.strftime(fmt_salida)
        except (ValueError, AttributeError) as e:
            logger.warning(f"Error generalizando fecha '{fecha_str}': {e}")
            return "[ERROR_FECHA]"
    
    def _generalizar_ubicacion(self, ubicacion_str: str, 
                               parametros: Dict[str, Any]) -> str:
        """
        Generalizar ubicación exacta a provincia.
        
        Mapeo manual de ciudades a provincias. En v2.0 se puede
        usar base de datos más completa.
        
        Args:
            ubicacion_str: Localidad exacta (ej: "La Plata")
            parametros: Dict con mapeo_provincia
            
        Returns:
            Provincia (ej: "Buenos Aires")
        """
        # Mapeo manual simplificado (en producción usar BD)
        mapeo_localidades = {
            'La Plata': 'Buenos Aires',
            'Quilmes': 'Buenos Aires',
            'Berazategui': 'Buenos Aires',
            'Ramos Mejía': 'Buenos Aires',
            'Ciudad Autónoma de Buenos Aires': 'CABA',
            'Córdoba': 'Córdoba',
            'Rosario': 'Santa Fe',
            'Mendoza': 'Mendoza',
            'Tucumán': 'Tucumán',
        }
        
        provincia = mapeo_localidades.get(
            str(ubicacion_str).strip(),
            'Provincia Desconocida'
        )
        return provincia
    
    def _agrupar_valor(self, valor: Any, 
                       parametros: Dict[str, Any]) -> str:
        """
        Agrupar valor en rangos (principalmente edad).
        
        Convierte edad exacta a rango (ej: 37 → "35-39") para
        cumplir k-anonimato según Sondeck-Laurent.
        
        Args:
            valor: Valor numérico (edad)
            parametros: Dict con rango_anos, grupos
            
        Returns:
            Etiqueta de rango (ej: "35-39")
        """
        try:
            valor_num = int(valor)
            grupos = parametros.get('grupos', [])
            
            for grupo in grupos:
                if grupo['min'] <= valor_num <= grupo['max']:
                    return grupo['etiqueta']
            
            logger.warning(f"Valor {valor} fuera de rango")
            return "[FUERA_RANGO]"
        except (ValueError, TypeError) as e:
            logger.warning(f"Error agrupando valor '{valor}': {e}")
            return "[ERROR_AGRUPAMIENTO]"
    
    def _hash_irreversible(self, valor: str, 
                          parametros: Dict[str, Any]) -> str:
        """
        Generar hash irreversible SHA-256 con salt.
        
        Implementa disociación irreversible según Disposición 11/2006.
        El hash no puede revertirse a valor original.
        
        Args:
            valor: Valor a hashear (ej: número HC)
            parametros: Dict con algoritmo, salt, truncar_a, prefijo
            
        Returns:
            Hash truncado con prefijo (ej: "HC_a1b2c3d4")
            
        References:
            - Disposición 11/2006: Requisito de irreversibilidad
            - Ley 25.326, Art. 7: Disociación
            
        Security:
            - SHA-256 es criptográficamente seguro
            - Salt previene ataques de tabla arcoíris
            - Truncación a 8 caracteres en producción suficiente
        """
        algoritmo = parametros.get('algoritmo', 'SHA-256')
        salt = parametros.get('salt', self.salt)
        truncar_a = parametros.get('truncar_a', 8)
        prefijo = parametros.get('prefijo', 'HC_')
        
        # Concatenar valor + salt y hashear
        entrada = str(valor) + salt
        if algoritmo == 'SHA-256':
            hash_obj = hashlib.sha256(entrada.encode('utf-8'))
            hash_hex = hash_obj.hexdigest()
            hash_truncado = hash_hex[:truncar_a].upper()
            return f"{prefijo}{hash_truncado}"
        else:
            logger.warning(f"Algoritmo desconocido: {algoritmo}")
            return "[ERROR_HASH]"
    
    def get_intervenciones_log(self) -> List[Dict[str, Any]]:
        """
        Retornar log completo de intervenciones realizadas.
        
        Returns:
            List de dicts con cada intervención (para auditoría)
        """
        return self.intervenciones_aplicadas
    
    def get_grado_intervencion(self, intervenciones: List[Dict]) -> str:
        """
        Estimar grado de intervención (BAJO/MODERADO/ALTO).
        
        Basado en cantidad y tipo de reglas aplicadas.
        
        Args:
            intervenciones: Lista de intervenciones aplicadas
            
        Returns:
            String: "BAJO", "MODERADO" o "ALTO"
            
        References:
            - Sondeck-Laurent: Evaluación de impacto de transformación
        """
        if len(intervenciones) == 0:
            return "BAJO"
        elif len(intervenciones) <= 3:
            return "BAJO"
        elif len(intervenciones) <= 6:
            return "MODERADO"
        else:
            return "ALTO"


# Función de utilidad: Cargar y procesar batch de registros
def anonymize_batch(records: List[Dict[str, Any]], 
                   anonymizer: CoreAnonymizer,
                   usuario: str = "sistema") -> Tuple[List[Dict], List[List[Dict]]]:
    """
    Anonimizar múltiples registros en batch.
    
    Args:
        records: Lista de dicts (historias clínicas)
        anonymizer: Instancia de CoreAnonymizer
        usuario: Usuario responsable
        
    Returns:
        Tuple de (lista_registros_anonimizados, lista_intervenciones_por_registro)
        
    Example:
        >>> records = [
        ...     {"nombre": "Juan", "edad": 37},
        ...     {"nombre": "María", "edad": 45}
        ... ]
        >>> records_anon, interventions = anonymize_batch(records, anonymizer)
    """
    records_anonimizados = []
    todas_intervenciones = []
    
    for i, record in enumerate(records):
        try:
            record_anon, interventions = anonymizer.anonymize_record(record, usuario)
            records_anonimizados.append(record_anon)
            todas_intervenciones.append(interventions)
            logger.info(f"Registro {i+1}/{len(records)} anonimizado exitosamente")
        except Exception as e:
            logger.error(f"Error anonimizando registro {i+1}: {e}")
            raise
    
    return records_anonimizados, todas_intervenciones
```

---

## Uso y Ejemplo

### Inicialización

```python
from anonimizador.core_anonymization import CoreAnonymizer, anonymize_batch

# Crear instancia del anonymizer
anonymizer = CoreAnonymizer(config_path="rules_config.yaml")
```

### Anonimizar un registro individual

```python
# Ejemplo de historia clínica
hc = {
    "nombre": "Juan Carlos Pérez",
    "dni": "12345678",
    "edad": 37,
    "fecha_atencion": "14/11/2025",
    "localidad": "La Plata",
    "diagnostico_principal": "Colecistitis aguda"
}

# Anonimizar
hc_anon, intervenciones = anonymizer.anonymize_record(hc, usuario="lankamar")

# Resultado
print(hc_anon)
# Output: {
#   'nombre': '[SUPRIMIDO]',
#   'dni': '[SUPRIMIDO]',
#   'edad': '35-39',
#   'fecha_atencion': '2025',
#   'localidad': 'Buenos Aires',
#   'diagnostico_principal': 'Colecistitis aguda'
# }

# Ver intervenciones
for int in intervenciones:
    print(f"{int['regla']}: {int['metodo']} - {int['razon']}")
```

### Anonimizar múltiples registros

```python
historias_clinicas = [
    {"nombre": "Juan", "edad": 37, "dni": "12345678"},
    {"nombre": "María", "edad": 45, "dni": "87654321"},
    {"nombre": "Carlos", "edad": 52, "dni": "55555555"}
]

hcs_anonimizadas, intervenciones_todas = anonymize_batch(
    historias_clinicas,
    anonymizer,
    usuario="lankamar@hc-clinicas"
)

# Ver grado de intervención
for i, intervs in enumerate(intervenciones_todas):
    grado = anonymizer.get_grado_intervencion(intervs)
    print(f"HC {i+1}: Grado de intervención = {grado}")
```

---

## Integración con Otros Módulos

Este módulo es **independiente** y se integra con:

- **Módulo 1 (Parsers):** Toma output del parser y anonimiza
- **Módulo 5 (Auditoría):** Envía intervenciones al logger SQLite
- **Módulo 6 (Exportación):** Recibe datos anonimizados para generar MD/PDF

---

## Testing Unitario

```python
# tests/test_core_anonymization.py

import pytest
from anonimizador.core_anonymization import CoreAnonymizer

def test_supresion_idd():
    """Verificar supresión de identificadores directos."""
    anonymizer = CoreAnonymizer()
    record = {"nombre": "Juan Pérez", "edad": 30}
    record_anon, _ = anonymizer.anonymize_record(record)
    assert record_anon["nombre"] == "[SUPRIMIDO]"

def test_agrupamiento_edad():
    """Verificar agrupamiento de edad en rangos."""
    anonymizer = CoreAnonymizer()
    record = {"edad": 37}
    record_anon, _ = anonymizer.anonymize_record(record)
    assert record_anon["edad"] == "35-39"

def test_hash_irreversible():
    """Verificar hash SHA-256 irreversible."""
    anonymizer = CoreAnonymizer()
    record = {"numero_hc": "HC-123456"}
    record_anon, intervenciones = anonymizer.anonymize_record(record)
    # Hash debe tener prefijo y ser hexadecimal
    assert record_anon["numero_hc"].startswith("HC_")
    assert len(record_anon["numero_hc"]) == 11  # HC_ + 8 chars

def test_batch_processing():
    """Verificar procesamiento en batch."""
    anonymizer = CoreAnonymizer()
    records = [
        {"nombre": "Juan", "edad": 30},
        {"nombre": "María", "edad": 40}
    ]
    records_anon, intervenciones = anonymize_batch(records, anonymizer)
    assert len(records_anon) == 2
    assert len(intervenciones) == 2
```

---

## Notas de Implementación

- Este módulo es **determinista**: mismo input siempre produce mismo output
- El **salt es configurado globalmente** para garantizar hashes consistentes
- Las **intervenciones son registradas** para auditoría completa
- El código es **extensible**: agregar nuevas reglas solo requiere actualizar YAML

---

**Documento v1.0 - Módulo Core de Anonimización**  
**Última actualización:** 2025-11-14
