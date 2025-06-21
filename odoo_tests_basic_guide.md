# 🧪 Guía Completa de Tests en Odoo 17/18 - Cheat Sheet

> 💡 **Pro Tip**: Siempre ejecuta los tests antes de hacer commit. Un test que falla en desarrollo es más fácil de arreglar que en producción.
> Usa mis videos de pre-commit de youtube para primero tener el codigo limpio y de calidad (Consejo de LisarDoo)

## 📁 Estructura base de Archivos en un modulo con tests

```
mi_modulo/
├── tests/
│   ├── __init__.py              # from . import test_*
│   ├── common.py                # Clases base comunes
│   ├── test_modelo.py           # Tests unitarios del modelo
│   ├── test_controlador.py      # Tests de controladores
│   └── test_integration.py      # Tests de integración
├── static/
│   └── tests/
│       ├── tours/
│       │   └── mi_tour.js       # Tours de integración
│       └── helpers/
│           └── test_utils.js    # Utilidades JS
└── data/
    └── tours.xml                # Onboarding tours data
```

## 📋 Tabla de Contenidos
1. [Introducción](#introducción)
2. [Tipos de Tests](#tipos-de-tests)
3. [Estructura de Archivos](#estructura-de-archivos)
4. [Clases Base y Decoradores](#clases-base-y-decoradores)
5. [Comandos de Ejecución](#comandos-de-ejecución)
6. [Ejemplos Básicos](#ejemplos-básicos)
7. [Ejemplo Completo: Campo Total](#ejemplo-completo-campo-total)
8. [Ejemplo Completo: Área Cuadrado](#ejemplo-completo-área-cuadrado)
9. [Tips y Mejores Prácticas](#tips-y-mejores-prácticas)
10. [Troubleshooting](#troubleshooting)

---

## 🎯 Introducción

Los tests en Odoo son fundamentales para garantizar la calidad del código y prevenir regresiones. Odoo ofrece tres tipos principales de testing que cubren desde la lógica de negocio hasta la integración completa del sistema.

**¿Por qué testear en Odoo?**
- ✅ Detectar bugs antes de producción
- ✅ Prevenir regresiones en funcionalidades existentes
- ✅ Documentar el comportamiento esperado
- ✅ Facilitar refactoring seguro
- ✅ Mejorar la confianza en deployments

---

## 🔧 Tipos de Tests

### 1. **Tests Python** 🐍
- **Propósito**: Testear lógica de negocio de modelos
- **Ubicación**: `mi_modulo/tests/`
- **Cuando usar**: Campos calculados, métodos, validaciones, workflows

### 2. **Tests JavaScript** ⚡
- **Propósito**: Testear código JS en aislamiento
- **Ubicación**: `mi_modulo/static/tests/`
- **Cuando usar**: Widgets, componentes, utilidades JS

### 3. **Tours (Tests de Integración)** 🎮
- **Propósito**: Simular flujos de usuario real
- **Ubicación**: `mi_modulo/static/tests/tours/`
- **Cuando usar**: Workflows completos, interacción frontend-backend

---

## 📁 Estructura de Archivos

```
mi_modulo/
├── __manifest__.py
├── models/
│   └── mi_modelo.py
├── views/
│   └── mi_vista.xml
├── tests/
│   ├── __init__.py          # ⚠️ IMPORTANTE: Importar tests aquí
│   ├── common.py            # Clases base comunes
│   ├── test_mi_modelo.py    # Tests del modelo
│   └── test_integration.py  # Tests de integración
├── static/
│   └── tests/
│       ├── mi_test.js       # Tests JavaScript
│       └── tours/
│           └── mi_tour.js   # Tours de integración
```

### ⚠️ Configuración Obligatoria

**En `tests/__init__.py`:**
```python
from . import test_mi_modelo
from . import test_integration
```

**En `__manifest__.py`:**
```python
{
    'name': 'Mi Módulo',
    # ... otros campos
    'test': True,  # Permite ejecutar tests
}
```

---

## 🏗️ Clases Base y Decoradores

### **Clases Base Principales**

```python
from odoo.tests.common import TransactionCase, SingleTransactionCase, tagged, users

# Para tests de modelos (más común)
class MiTest(TransactionCase):
    pass

# Para tests HTTP/frontend
from odoo.tests.common import HttpCase
class MiTestHTTP(HttpCase):
    pass
```

| Clase | Uso | Características |
|-------|-----|----------------|
| `TransactionCase` | Tests de modelos | Cada test en su propia transacción |
| `SingleTransactionCase` | Tests compartidos | Todos los tests comparten transacción |
| `HttpCase` | Tests frontend | Incluye servidor HTTP |

### **Decoradores Esenciales**

```python
@tagged('mi_modulo', 'post_install')
@users('admin')
def test_mi_funcionalidad(self):
    pass
```

| Decorador | Propósito | Ejemplo |
|-----------|-----------|---------|
| `@tagged()` | Agrupar/filtrar tests | `@tagged('crm', 'slow')` |
| `@users()` | Usuario específico | `@users('admin')` |
| `@mute_logger()` | Silenciar logs | `@mute_logger('odoo.sql_db')` |

### **Tags Especiales**

- `standard`: Tests que se ejecutan por defecto
- `at_install`: Se ejecuta al instalar el módulo
- `post_install`: Se ejecuta después de instalar todos los módulos
- `-at_install`: NO se ejecuta al instalar

---

## 🚀 Comandos de Ejecución

### **Comandos Básicos**

```bash
# Ejecutar todos los tests del módulo
./odoo-bin -c config.conf -d test_db -i mi_modulo --test-enable --stop-after-init

# Tests específicos por tag
./odoo-bin -c config.conf -d test_db -i mi_modulo --test-enable --test-tags=mi_tag

# Test específico
./odoo-bin -c config.conf -d test_db -i mi_modulo --test-enable --test-tags=/mi_modulo:MiClaseTest.test_mi_metodo

# Tests excluyendo algunos
./odoo-bin -c config.conf -d test_db -i mi_modulo --test-enable --test-tags='standard,-slow'
```

### **Opciones Útiles**

| Opción | Descripción |
|--------|-------------|
| `--test-enable` | Habilita tests |
| `--test-tags` | Filtra por tags |
| `--stop-after-init` | Para después de tests |
| `-i modulo` | Instala módulo |
| `-u modulo` | Actualiza módulo |

---

## 📚 Ejemplos Básicos

### **Ejemplo 1: Test Simple de Modelo**

```python
# tests/test_basico.py
from odoo.tests.common import TransactionCase

class TestBasico(TransactionCase):
    
    def setUp(self):
        super().setUp()
        self.partner = self.env['res.partner'].create({
            'name': 'Test Partner',
            'email': 'test@example.com'
        })
    
    def test_partner_creation(self):
        """Test que verifica creación de partner"""
        self.assertTrue(self.partner.exists())
        self.assertEqual(self.partner.name, 'Test Partner')
        self.assertEqual(self.partner.email, 'test@example.com')
    
    def test_partner_display_name(self):
        """Test que verifica display_name"""
        expected_name = 'Test Partner'
        self.assertEqual(self.partner.display_name, expected_name)
```

### **Ejemplo 2: Test con Validaciones**

```python
# tests/test_validaciones.py
from odoo.tests.common import TransactionCase
from odoo.exceptions import ValidationError

class TestValidaciones(TransactionCase):
    
    def test_email_validation(self):
        """Test que verifica validación de email"""
        with self.assertRaises(ValidationError):
            self.env['res.partner'].create({
                'name': 'Test Partner',
                'email': 'email_invalido'  # Email inválido
            })
    
    def test_required_field(self):
        """Test que verifica campo requerido"""
        with self.assertRaises(ValidationError):
            self.env['res.partner'].create({
                # Falta el campo 'name' que es requerido
                'email': 'test@example.com'
            })
```

---

## 🎯 Ejemplo Completo: Campo Total

Este ejemplo crea una herencia del CRM Lead con un campo calculado "total" que suma dos campos.

### **1. Archivo: `__manifest__.py`**

```python
{
    'name': 'CRM Lead Total',
    'version': '18.0.1.0.0',
    'category': 'Sales/CRM',
    'summary': 'Agrega campo total calculado a CRM Leads',
    'depends': ['crm'],
    'data': [
        'views/crm_lead_views.xml',
    ],
    'demo': [],
    'installable': True,
    'auto_install': False,
    'application': False,
}
```

### **2. Archivo: `models/__init__.py`**

```python
from . import crm_lead
```

### **3. Archivo: `models/crm_lead.py`**

```python
from odoo import api, fields, models

class CrmLead(models.Model):
    _inherit = 'crm.lead'
    
    # Campos para la suma
    campo_a = fields.Float(
        string='Campo A',
        default=0.0,
        help='Primer valor para el cálculo del total'
    )
    
    campo_b = fields.Float(
        string='Campo B', 
        default=0.0,
        help='Segundo valor para el cálculo del total'
    )
    
    # Campo calculado
    total = fields.Float(
        string='Total',
        compute='_compute_total',
        store=True,
        help='Suma de Campo A + Campo B'
    )
    
    @api.depends('campo_a', 'campo_b')
    def _compute_total(self):
        """Calcula el total como suma de campo_a + campo_b"""
        for record in self:
            record.total = record.campo_a + record.campo_b
```

### **4. Archivo: `views/crm_lead_views.xml`**

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Vista lista heredada -->
    <record id="view_crm_lead_tree_inherit" model="ir.ui.view">
        <field name="name">crm.lead.tree.inherit</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='expected_revenue']" position="after">
                <field name="campo_a"/>
                <field name="campo_b"/>
                <field name="total"/>
            </xpath>
        </field>
    </record>

    <!-- Vista formulario heredada -->
    <record id="view_crm_lead_form_inherit" model="ir.ui.view">
        <field name="name">crm.lead.form.inherit</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='expected_revenue']" position="after">
                <field name="campo_a"/>
                <field name="campo_b"/>
                <field name="total" readonly="1"/>
            </xpath>
        </field>
    </record>
</odoo>
```

### **5. Archivo: `tests/__init__.py`**

```python
from . import test_crm_lead_total
```

### **6. Archivo: `tests/test_crm_lead_total.py`**

```python
from odoo.tests.common import TransactionCase

class TestCrmLeadTotal(TransactionCase):
    
    def setUp(self):
        super().setUp()
        self.lead = self.env['crm.lead'].create({
            'name': 'Test Lead Total',
            'campo_a': 10.0,
            'campo_b': 20.0,
        })
    
    def test_total_calculation(self):
        """Test que verifica el cálculo del campo total"""
        self.assertEqual(self.lead.total, 30.0)
        
    def test_total_recalculation_campo_a(self):
        """Test que verifica recálculo al cambiar campo_a"""
        self.lead.campo_a = 15.0
        self.assertEqual(self.lead.total, 35.0)
    
    def test_total_recalculation_campo_b(self):
        """Test que verifica recálculo al cambiar campo_b"""
        self.lead.campo_b = 25.0
        self.assertEqual(self.lead.total, 35.0)
    
    def test_total_with_zeros(self):
        """Test que verifica cálculo con valores cero"""
        lead_zero = self.env['crm.lead'].create({
            'name': 'Test Lead Zero',
            'campo_a': 0.0,
            'campo_b': 0.0,
        })
        self.assertEqual(lead_zero.total, 0.0)
    
    def test_total_with_negative_values(self):
        """Test que verifica cálculo con valores negativos"""
        lead_negative = self.env['crm.lead'].create({
            'name': 'Test Lead Negative',
            'campo_a': -5.0,
            'campo_b': 10.0,
        })
        self.assertEqual(lead_negative.total, 5.0)
```

---

## 📐 Ejemplo Completo: Área Cuadrado

Este ejemplo agrega un campo calculado para el área de un cuadrado.

### **1. Archivo: `models/crm_lead.py` (Agregar al anterior)**

```python
from odoo import api, fields, models
import math

class CrmLead(models.Model):
    _inherit = 'crm.lead'
    
    # ... campos anteriores ...
    
    # Campo para lado del cuadrado
    lado_cuadrado = fields.Float(
        string='Lado del Cuadrado',
        default=0.0,
        help='Longitud del lado del cuadrado en metros'
    )
    
    # Campo calculado para área
    area_cuadrado = fields.Float(
        string='Área del Cuadrado',
        compute='_compute_area_cuadrado',
        store=True,
        help='Área del cuadrado en metros cuadrados'
    )
    
    @api.depends('lado_cuadrado')
    def _compute_area_cuadrado(self):
        """Calcula el área del cuadrado como lado²"""
        for record in self:
            if record.lado_cuadrado >= 0:
                record.area_cuadrado = record.lado_cuadrado ** 2
            else:
                record.area_cuadrado = 0.0
```

### **2. Archivo: `views/crm_lead_views.xml` (Actualizar)**

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Vista lista heredada -->
    <record id="view_crm_lead_tree_inherit" model="ir.ui.view">
        <field name="name">crm.lead.tree.inherit</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='expected_revenue']" position="after">
                <field name="campo_a"/>
                <field name="campo_b"/>
                <field name="total"/>
                <field name="lado_cuadrado"/>
                <field name="area_cuadrado"/>
            </xpath>
        </field>
    </record>

    <!-- Vista formulario heredada -->
    <record id="view_crm_lead_form_inherit" model="ir.ui.view">
        <field name="name">crm.lead.form.inherit</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='expected_revenue']" position="after">
                <group string="Cálculos Personalizados">
                    <group>
                        <field name="campo_a"/>
                        <field name="campo_b"/>
                        <field name="total" readonly="1"/>
                    </group>
                    <group>
                        <field name="lado_cuadrado"/>
                        <field name="area_cuadrado" readonly="1"/>
                    </group>
                </group>
            </xpath>
        </field>
    </record>
</odoo>
```

### **3. Archivo: `tests/__init__.py` (Actualizar)**

```python
from . import test_crm_lead_total
from . import test_crm_lead_area
```

### **4. Archivo: `tests/test_crm_lead_area.py`**

```python
from odoo.tests.common import TransactionCase

class TestCrmLeadArea(TransactionCase):
    
    def setUp(self):
        super().setUp()
        self.lead = self.env['crm.lead'].create({
            'name': 'Test Lead Area',
            'lado_cuadrado': 5.0,
        })
    
    def test_area_calculation(self):
        """Test que verifica el cálculo del área del cuadrado"""
        self.assertEqual(self.lead.area_cuadrado, 25.0)
    
    def test_area_recalculation(self):
        """Test que verifica recálculo del área al cambiar el lado"""
        self.lead.lado_cuadrado = 10.0
        self.assertEqual(self.lead.area_cuadrado, 100.0)
    
    def test_area_with_zero(self):
        """Test que verifica área con lado cero"""
        lead_zero = self.env['crm.lead'].create({
            'name': 'Test Lead Zero',
            'lado_cuadrado': 0.0,
        })
        self.assertEqual(lead_zero.area_cuadrado, 0.0)
    
    def test_area_with_negative_side(self):
        """Test que verifica área con lado negativo"""
        lead_negative = self.env['crm.lead'].create({
            'name': 'Test Lead Negative',
            'lado_cuadrado': -5.0,
        })
        self.assertEqual(lead_negative.area_cuadrado, 0.0)
    
    def test_area_with_decimal(self):
        """Test que verifica área con valores decimales"""
        lead_decimal = self.env['crm.lead'].create({
            'name': 'Test Lead Decimal',
            'lado_cuadrado': 2.5,
        })
        self.assertEqual(lead_decimal.area_cuadrado, 6.25)
```

---

## 💡 Tips y Mejores Prácticas

### **🎯 Estructura de Tests**

```python
class TestMiModelo(TransactionCase):
    
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Datos que no cambian entre tests
        cls.user_admin = cls.env.ref('base.user_admin')
    
    def setUp(self):
        super().setUp()
        # Datos específicos para cada test
        self.partner = self.env['res.partner'].create({
            'name': 'Test Partner'
        })
    
    def test_something(self):
        # Arrange (preparar)
        # Act (ejecutar)
        # Assert (verificar)
        pass
```

### **✅ Assertions Más Usadas**

```python
# Comparaciones básicas
self.assertEqual(a, b, "Mensaje de error")
self.assertNotEqual(a, b)
self.assertTrue(condition)
self.assertFalse(condition)

# Específicas de Odoo
self.assertRecordValues(records, [{'field': value}])

# Para excepciones
with self.assertRaises(ValidationError):
    # código que debe fallar
    pass

# Contenido
self.assertIn(item, container)
self.assertNotIn(item, container)
```

### **🔧 Environment y Contexto**

```python
# Environment básico
self.env['model.name']

# Con usuario específico
self.env['model.name'].with_user(self.user_admin)

# Con contexto
self.env['model.name'].with_context(lang='es_ES')

# Referencias XML
self.env.ref('module.xml_id')
```

### **📝 Nomenclatura**

- **Métodos de test**: `test_descripcion_clara`
- **Clases de test**: `TestNombreModelo`
- **Archivos de test**: `test_nombre_modelo.py`
- **Tags**: usa nombres descriptivos

---

## 🚨 Troubleshooting

### **Problemas Comunes**

| Problema | Causa | Solución |
|----------|-------|----------|
| Tests no se ejecutan | No importados en `__init__.py` | Agregar import |
| `AttributeError` en modelo | Modelo no cargado | Verificar dependencias |
| Tests fallan aleatoriamente | Datos compartidos entre tests | Usar `setUp()` correctamente |
| `TransactionRollbackError` | Transacción corrupta | Verificar commits/rollbacks |

### **Debug de Tests**

```python
# Agregar prints para debug
def test_mi_funcionalidad(self):
    print(f"Partner: {self.partner}")
    print(f"Total: {self.partner.total}")
    # ... resto del test

# Usar pdb para debug interactivo
import pdb; pdb.set_trace()
```

### **Tests Lentos**

```python
# Usar @tagged para clasificar
@tagged('slow')
def test_operacion_lenta(self):
    pass

# Ejecutar sin tests lentos
# --test-tags='standard,-slow'
```

---

## 🎓 Comandos de Referencia Rápida

```bash
# Tests básicos
./odoo-bin -c config.conf -d test_db -i mi_modulo --test-enable --stop-after-init

# Solo tests post-install
./odoo-bin -c config.conf -d test_db -i mi_modulo --test-enable --test-tags=post_install

# Test específico
./odoo-bin -c config.conf -d test_db -i mi_modulo --test-enable --test-tags=/mi_modulo:TestClase.test_metodo

# Con logs SQL para performance
./odoo-bin -c config.conf -d test_db -i mi_modulo --test-enable --log-sql --stop-after-init
```

---

## 🚀 ¡A Practicar!

1. **Copia y pega** los ejemplos en tu módulo custom
2. **Ejecuta los tests** para verificar que funcionan
3. **Modifica los valores** y observa cómo fallan los tests
4. **Agrega tus propios tests** basándote en estos ejemplos

¡Recuerda: Un buen test es específico, rápido y confiable! 🎯


### Fuentes

[Doc. Oficial](https://www.odoo.com/documentation/18.0/developer/reference/backend/testing.html)


**Autor**: JC Montoya - [camuscleverit.es](https://camuscleverit.es) | [YouTube](https://YouTube.com/c/jcmontoya) |  [LisarDoo](www.youtube.com/@lisardooEDU)
