# 🧪 Guía Completa de Tests en Odoo 17/18 - Cheat Sheet

## 📁 Estructura de Archivos

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

## 🐍 Tests Python

### Clases Base Principales

```python
from odoo.tests.common import (
    TransactionCase,        # Rollback automático por test
    SingleTransactionCase,  # Misma transacción para todos
    SavepointCase,         # Con savepoints
    HttpCase,              # Para tests con servidor HTTP
    tagged, users
)

# Herencia más común
from odoo.addons.mi_modulo.tests.common import TestMiModuloCommon
```

### Decoradores Esenciales

```python
@tagged('standard', 'at_install')     # Tags por defecto
@tagged('-standard', 'post_install')  # Excluir standard
@tagged('slow', 'external')           # Tags personalizados
@users('admin')                       # Usuario específico
@mute_logger('odoo.addons.base.models.ir_model')  # Silenciar logs
@warmup                               # Pre-calentar cache
```

### Template Básico de Test

```python
# -*- coding: utf-8 -*-
from odoo.tests.common import TransactionCase, tagged, users
from odoo.exceptions import ValidationError, AccessError


@tagged('mi_modulo')
class TestMiModelo(TransactionCase):
    
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Datos una sola vez para toda la clase
        cls.partner = cls.env['res.partner'].create({
            'name': 'Test Partner',
            'email': 'test@example.com'
        })
    
    def setUp(self):
        super().setUp()
        # Configuración antes de cada test
        
    @users('admin')
    def test_algo_importante(self):
        """Test que algo funcione correctamente"""
        # Arrange
        record = self.env['mi.modelo'].create({'campo': 'valor'})
        
        # Act
        result = record.mi_metodo()
        
        # Assert
        self.assertEqual(result, 'valor_esperado')
        self.assertTrue(record.campo_booleano)
    
    def test_validacion_error(self):
        """Test que las validaciones funcionen"""
        with self.assertRaises(ValidationError):
            self.env['mi.modelo'].create({'campo_requerido': False})
```

### Assertions Más Utilizadas

```python
# Comparaciones básicas
self.assertEqual(a, b)
self.assertNotEqual(a, b)
self.assertTrue(condition)
self.assertFalse(condition)
self.assertIn(item, container)
self.assertNotIn(item, container)

# Específicas de Odoo
self.assertRecordValues(records, [
    {'field1': value1, 'field2': value2},
    {'field1': value3, 'field2': value4}
])

# Para excepciones
with self.assertRaises(ValidationError) as cm:
    # código que debe fallar
    pass
self.assertIn('mensaje esperado', str(cm.exception))

# Performance
with self.assertQueryCount(5):
    # operación que debe hacer exactamente 5 queries
    pass
```

### Environment y Referencias

```python
# Environment básico
self.env['model.name']
self.env.user                    # Usuario actual
self.env.company                 # Compañía actual
self.env.context                 # Contexto actual

# Con usuario/contexto específico
self.env['model.name'].with_user(user_id)
self.env['model.name'].with_context(lang='es_ES')
self.env['model.name'].sudo()

# Referencias XML
self.env.ref('module.xml_id')
self.browse_ref('module.xml_id')  # Solo en test classes
```

## 🧪 Form Helper para Tests de Vistas

```python
from odoo.tests import Form

def test_form_view(self):
    """Test comportamiento de formulario"""
    with Form(self.env['sale.order']) as form:
        form.partner_id = self.partner
        form.order_line.new().product_id = self.product
        form.order_line[0].product_uom_qty = 5
        
        # Simular onchange
        form.order_line[0].price_unit = 100
        
        order = form.save()
        
    self.assertEqual(order.amount_total, 500)
```

## 🌐 Tests JavaScript/QUnit

### Estructura Básica

```javascript
// static/tests/mi_test.js
odoo.define('mi_modulo.tests', function (require) {
"use strict";

var testUtils = require('web.test_utils');
var FormView = require('web.FormView');

QUnit.module('MiModulo', {
    beforeEach: function () {
        this.data = {
            'partner': {
                fields: {
                    name: {string: 'Name', type: 'char'},
                },
                records: [
                    {id: 1, name: 'First Partner'},
                ]
            }
        };
    }
}, function () {
    
    QUnit.test('basic form view test', async function (assert) {
        assert.expect(2);
        
        var form = await testUtils.createView({
            View: FormView,
            model: 'partner',
            data: this.data,
            arch: '<form><field name="name"/></form>',
            res_id: 1,
        });
        
        assert.containsOnce(form, '.o_field_widget[name="name"]');
        assert.strictEqual(form.$('.o_field_widget[name="name"]').val(), 'First Partner');
        
        form.destroy();
    });
});

});
```

### Helpers de Test Utils

```javascript
// Creación de vistas
testUtils.createView({
    View: FormView,
    model: 'partner',
    data: this.data,
    arch: '<form>...</form>',
    res_id: 1,
    mockRPC: function (route, args) {
        // Mock RPCs
    }
});

// DOM helpers
testUtils.dom.click(target);
testUtils.dom.triggerEvents(target, ['focus', 'blur']);
testUtils.fields.editInput(target, 'new value');

// Mock server
testUtils.mock.addMockEnvironment();
```

### Assertions QUnit + Odoo

```javascript
// QUnit básicas
assert.ok(value);
assert.equal(actual, expected);
assert.strictEqual(actual, expected);
assert.deepEqual(actual, expected);

// Odoo específicas
assert.containsOnce(widget, '.css-selector');
assert.containsN(widget, '.css-selector', 3);
assert.containsNone(widget, '.css-selector');
assert.hasClass(widget, 'css-class');
assert.doesNotHaveClass(widget, 'css-class');
assert.isVisible(widget);
assert.isNotVisible(widget);
```

## 🚗 Tours (Integración)

### Tour de Test Básico

```javascript
// static/tests/tours/mi_tour.js
import tour from 'web_tour.tour';

tour.register('mi_tour_test', {
    test: true,
    url: '/web',
}, [
    tour.stepUtils.showAppsMenuItem(),
    {
        trigger: '.o_app[data-menu-xmlid="mi_modulo.menu_root"]',
        content: 'Abrir mi módulo',
        run: "click",
    },
    {
        trigger: '.o_list_button_add',
        content: 'Crear nuevo registro',
        run: "click",
    },
    {
        trigger: '.o_field_widget[name="name"] input',
        content: 'Llenar nombre',
        run: "edit Nombre Test",
    },
    {
        trigger: '.o_form_button_save',
        content: 'Guardar',
        run: "click",
    },
    {
        trigger: '.o_form_saved',
        content: 'Verificar guardado',
        run: function() {
            // Verificaciones adicionales
        },
    }
]);
```

### Helpers de Tour

```javascript
// Acciones disponibles en run:
run: "click"
run: "edit contenido"
run: "fill contenido"
run: "clear"
run: "select value"
run: "check"    // checkbox
run: "uncheck"  // checkbox
run: "hover"
run: "drag_and_drop .target"

// Función personalizada
async run(helpers) {
    helpers.click();
    await helpers.waitFor('.elemento');
}
```

### Tour de Onboarding

```javascript
// Para onboarding tours
tour.register('mi_onboarding', {
    rainbowMan: true,
    sequence: 10,
}, [
    {
        trigger: '.o_onboarding_step',
        content: 'Primer paso del onboarding',
        position: 'bottom',
        run: "click",
    }
]);
```

### Test Python que ejecuta Tour

```python
from odoo.tests import HttpCase, tagged

@tagged('post_install', '-at_install')
class TestTours(HttpCase):
    
    def test_mi_tour(self):
        """Test del tour completo"""
        # Setup opcional
        self.env['mi.modelo'].create({'name': 'Setup Data'})
        
        # Ejecutar tour
        self.start_tour("/web", 'mi_tour_test', login="admin")
        
        # Verificaciones post-tour
        record = self.env['mi.modelo'].search([('name', '=', 'Nombre Test')])
        self.assertTrue(record.exists())
```

## ⚡ Comandos de Ejecución

### Comandos Básicos

```bash
# Todos los tests del módulo
./odoo-bin -c config.conf -d test_db -i mi_modulo --test-enable --stop-after-init

# Por tags
./odoo-bin -c config.conf -d test_db --test-tags=mi_modulo

# Tags específicos
./odoo-bin -c config.conf -d test_db --test-tags=standard,-slow

# Test específico
./odoo-bin -c config.conf -d test_db --test-tags=/mi_modulo:TestClass.test_method

# Con coverage
./odoo-bin -c config.conf -d test_db --test-tags=mi_modulo --test-enable
```

### Flags Útiles para Desarrollo

```bash
# Debug tours (abre browser)
--test-tags=mi_tour --browser-js-args="--start-maximized" --debug

# Screenshots on failure
--screenshots /tmp/test_screenshots

# Log SQL queries
--log-sql

# Solo tests post_install
--test-tags=post_install

# Excluir tests específicos
--test-tags=standard,-slow,-external
```

## 🛠️ Debugging y Tips

### Debug Tours en Browser

```javascript
// En consola del browser
odoo.startTour("nombre_tour");

// Con debug
odoo.startTour("nombre_tour", {debug: true});

// Pausar en step específico
{
    trigger: '.mi-elemento',
    break: true,  // Breakpoint aquí
    run: "click",
}

// Pausa manual
{
    trigger: '.mi-elemento',
    pause: true,  // Escribe play(); para continuar
    run: "click",
}
```

### Mock RPCs en Tests JS

```javascript
mockRPC: function (route, args) {
    if (args.method === 'search_read') {
        return Promise.resolve([
            {id: 1, name: 'Mocked Record'}
        ]);
    }
    return this._super.apply(this, arguments);
}
```

### Datos de Test Realistas

```python
@classmethod
def setUpClass(cls):
    super().setUpClass()
    
    # Configurar país para evitar sanitización de teléfonos
    cls.env.company.country_id = cls.env.ref('base.es')
    
    # Partner con datos completos
    cls.partner = cls.env['res.partner'].create({
        'name': 'Test Company S.L.',
        'is_company': True,
        'vat': 'ESB12345678',
        'email': 'info@testcompany.com',
        'phone': '+34912345678',
        'street': 'Calle Test 123',
        'city': 'Madrid',
        'zip': '28001',
        'country_id': cls.env.ref('base.es').id,
    })
```

## 📊 Best Practices

### Tests Python
- ✅ Un test por funcionalidad específica
- ✅ Nombres descriptivos: `test_should_validate_when_required_fields_present`
- ✅ AAA Pattern: Arrange, Act, Assert
- ✅ Cleanup con `tearDown()` si es necesario
- ✅ Usar `with_context()` para tests multiidioma
- ✅ Tests de validaciones con `assertRaises`

### Tests JavaScript
- ✅ Destruir widgets: `widget.destroy()`
- ✅ `assert.expect(n)` al inicio
- ✅ Mock RPCs para evitar llamadas reales
- ✅ `subTest()` para tests parametrizados

### Tours
- ✅ Último step debe dejar estado estable
- ✅ Usar `content` descriptivo para debug
- ✅ `isActive` para diferentes entornos
- ✅ Timeouts apropiados para elements lentos

## 🏷️ Tags Importantes

```python
# Por defecto en BaseCase
@tagged('standard', 'at_install')

# Tests lentos o externos
@tagged('-standard', 'slow')
@tagged('-standard', 'external')

# Post instalación (tours HTTP)
@tagged('-at_install', 'post_install')

# Por funcionalidad
@tagged('mi_modulo', 'ventas', 'crm')
@tagged('accounting', 'l10n')

# Performance
@tagged('performance')
```

## 🔧 Assets para JS Tests

```xml
<!-- En __manifest__.py -->
'assets': {
    'web.assets_tests': [
        'mi_modulo/static/tests/**/*.js',
    ],
    'web.qunit_suite_tests': [
        'mi_modulo/static/tests/tours/*.js',
    ],
}
```

## 📈 Performance Testing

```python
# Query count
with self.assertQueryCount(5):
    records = self.env['mi.modelo'].search([])
    records.mapped('computed_field')

# Timing
import time
start = time.time()
# operación
duration = time.time() - start
self.assertLess(duration, 1.0)  # Menos de 1 segundo
```

---

## 🚨 Errores Comunes y Soluciones

| Error | Causa | Solución |
|-------|-------|----------|
| Test no se ejecuta | No importado en `__init__.py` | Añadir `from . import test_file` |
| Tour falla random | Race conditions | Usar triggers más específicos |
| JS test no encuentra elemento | DOM no renderizado | `await testUtils.nextTick()` |
| Access Error en test | Usuario sin permisos | `@users('admin')` o `.sudo()` |
| ValidationError no se ejecuta | Datos válidos por defecto | Crear datos inválidos explícitamente |

---

> 💡 **Pro Tip**: Siempre ejecuta los tests antes de hacer commit. Un test que falla en desarrollo es más fácil de arreglar que en producción.

### Fuentes

[Doc. Oficial](https://www.odoo.com/documentation/18.0/developer/reference/backend/testing.html)


**Autor**: JC Montoya - [camuscleverit.es](https://camuscleverit.es) | [YouTube](https://YouTube.com/c/jcmontoya) |  [LisarDoo](www.youtube.com/@lisardooEDU)
