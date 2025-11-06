# Proyecto: Computadoras y accesorios
Lenguaje: Python  
Framework: Django  
Editor: VS Code

---

## Estructura de carpetas (inicial)
```
UIII_Computadoras_1291/
├─ .venv/
├─ backend_Computadoras/
│  ├─ backend_Computadoras/
│  │  ├─ __init__.py
│  │  ├─ settings.py
│  │  ├─ urls.py
│  │  └─ wsgi.py
│  └─ manage.py
└─ app_Computadoras/
   ├─ migrations/
   ├─ templates/
   │  ├─ base.html
   │  ├─ header.html
   │  ├─ navbar.html
   │  ├─ footer.html
   │  ├─ inicio.html
   │  └─ producto/
   │     ├─ agregar_producto.html
   │     ├─ ver_productos.html
   │     ├─ actualizar_producto.html
   │     └─ borrar_producto.html
   ├─ admin.py
   ├─ apps.py
   ├─ models.py
   ├─ views.py
   ├─ urls.py
   └─ __init__.py
```

---

## 1. Procedimiento para crear carpeta del Proyecto: `UIII_Computadoras_1291`
```bash
mkdir UIII_Computadoras_1291
cd UIII_Computadoras_1291
```

---

## 2. Procedimiento para abrir VS Code sobre la carpeta
```bash
code .
```
O usar File → Open Folder... en VS Code y seleccionar `UIII_Computadoras_1291`.

---

## 3. Procedimiento para abrir terminal en VS Code
En VS Code: `Terminal` → `New Terminal`. (Atajo: `Ctrl+ñ` en muchas distribuciones).

---

## 4. Crear carpeta entorno virtual `.venv` desde terminal de VS Code
Windows (PowerShell):
```powershell
python -m venv .venv
```
Linux / macOS:
```bash
python3 -m venv .venv
```

---

## 5. Activar el entorno virtual
Windows PowerShell:
```powershell
.venv\Scripts\Activate.ps1
```
Windows CMD:
```cmd
.venv\Scripts\activate
```
Linux / macOS:
```bash
source .venv/bin/activate
```

---

## 6. Activar intérprete de Python en VS Code
Abrir paleta: `Ctrl+Shift+P` → `Python: Select Interpreter` → seleccionar el intérprete que apunta a `.venv`.

---

## 7. Instalar Django
```bash
pip install django
# opcionalmente fijar versión
pip install "django==4.2"
```

---

## 8. Crear proyecto `backend_Computadoras` sin duplicar carpeta
```bash
django-admin startproject backend_Computadoras .
```

---

## 9. Ejecutar servidor en el puerto 8018
```bash
python manage.py runserver 8018
```

---

## 10. Copiar y pegar el link en el navegador
Abrir en el navegador:
```
http://127.0.0.1:8018/
```

---

## 11. Crear aplicación `app_Computadoras`
```bash
python manage.py startapp app_Computadoras
```

---

## 12. models.py (coloca en `app_Computadoras/models.py`)
```python
from django.db import models

# ==========================================
# MODELO: PROVEEDOR
# ==========================================
class Proveedor(models.Model):
    id = models.IntegerField(primary_key=True)
    nombre = models.CharField(max_length=255)
    contacto = models.CharField(max_length=255, blank=True, null=True)
    telefono = models.CharField(max_length=20, blank=True, null=True)
    correo = models.CharField(max_length=255, blank=True, null=True)
    direccion = models.CharField(max_length=255, blank=True, null=True)
    pais = models.CharField(max_length=100, blank=True, null=True)

    def __str__(self):
        return self.nombre

# ==========================================
# MODELO: PRODUCTO
# ==========================================
class Producto(models.Model):
    id = models.IntegerField(primary_key=True)
    nombre = models.CharField(max_length=255)
    descripcion = models.TextField(blank=True, null=True)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField()
    categoria = models.IntegerField()  # ID de categoría
    proveedor = models.ForeignKey(Proveedor, on_delete=models.CASCADE, related_name='productos')

    def __str__(self):
        return self.nombre

# ==========================================
# MODELO: FACTURA
# ==========================================
class Factura(models.Model):
    id = models.IntegerField(primary_key=True)
    fecha = models.DateField()
    total = models.DecimalField(max_digits=10, decimal_places=2)
    metodo_pago = models.CharField(max_length=100)
    cliente_id = models.IntegerField()
    empleado = models.CharField(max_length=255, blank=True, null=True)
    observaciones = models.TextField(blank=True, null=True)

    def __str__(self):
        return f"Factura #{self.id} - {self.fecha}"
```

> **Nota:** Django crea automáticamente un campo `id` si no lo defines. Definir `id = IntegerField(primary_key=True)` es válido pero obliga a pasar `id` al crear objetos.

---

## 12.5 Procedimiento para realizar las migraciones
```bash
python manage.py makemigrations app_Computadoras
python manage.py migrate
```

---

## 13. Trabajamos con el MODELO: PRODUCTO
(Se implementan vistas y plantillas para CRUD de `Producto`).

---

## 14. views.py (coloca en `app_Computadoras/views.py`)
```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Producto, Proveedor
from django.utils import timezone

def inicio_computadoras(request):
    contexto = {'hoy': timezone.now()}
    return render(request, 'inicio.html', contexto)

def agregar_producto(request):
    if request.method == 'POST':
        prod_id = request.POST.get('id')
        nombre = request.POST.get('nombre')
        descripcion = request.POST.get('descripcion')
        precio = request.POST.get('precio') or 0
        stock = request.POST.get('stock') or 0
        categoria = request.POST.get('categoria') or 0
        proveedor_id = request.POST.get('proveedor')

        proveedor = None
        if proveedor_id:
            try:
                proveedor = Proveedor.objects.get(id=int(proveedor_id))
            except Proveedor.DoesNotExist:
                proveedor = None

        prod_data = {
            'nombre': nombre,
            'descripcion': descripcion,
            'precio': precio,
            'stock': stock,
            'categoria': categoria,
            'proveedor': proveedor
        }
        if prod_id:
            Producto.objects.update_or_create(id=int(prod_id), defaults=prod_data)
        else:
            Producto.objects.create(**{k: v for k, v in prod_data.items() if v is not None})

        return redirect('ver_productos')
    else:
        proveedores = Proveedor.objects.all()
        return render(request, 'producto/agregar_producto.html', {'proveedores': proveedores})

def ver_productos(request):
    productos = Producto.objects.all()
    return render(request, 'producto/ver_productos.html', {'productos': productos})

def actualizar_producto(request, producto_id):
    producto = get_object_or_404(Producto, id=producto_id)
    proveedores = Proveedor.objects.all()
    return render(request, 'producto/actualizar_producto.html', {'producto': producto, 'proveedores': proveedores})

def realizar_actualizacion_producto(request, producto_id):
    producto = get_object_or_404(Producto, id=producto_id)
    if request.method == 'POST':
        producto.nombre = request.POST.get('nombre')
        producto.descripcion = request.POST.get('descripcion')
        producto.precio = request.POST.get('precio') or 0
        producto.stock = request.POST.get('stock') or 0
        producto.categoria = request.POST.get('categoria') or 0

        proveedor_id = request.POST.get('proveedor')
        if proveedor_id:
            try:
                producto.proveedor = Proveedor.objects.get(id=int(proveedor_id))
            except Proveedor.DoesNotExist:
                producto.proveedor = None

        producto.save()
        return redirect('ver_productos')
    return redirect('actualizar_producto', producto_id=producto_id)

def borrar_producto(request, producto_id):
    producto = get_object_or_404(Producto, id=producto_id)
    if request.method == 'POST':
        producto.delete()
        return redirect('ver_productos')
    return render(request, 'producto/borrar_producto.html', {'producto': producto})
```

---

## 15. Crear carpeta `templates` dentro de `app_Computadoras`
Ruta: `app_Computadoras/templates/`

---

## 16. Archivos HTML en `templates`

### `base.html`
```html
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>{% block title %}Sistema de Administración - Computadoras{% endblock %}</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
  <style>
    body { padding-top: 70px; padding-bottom: 70px; background-color: #f7fafc; }
    footer { position: fixed; bottom: 0; width: 100%; background: #ffffff; border-top: 1px solid #e5e7eb; }
  </style>
  {% block head %}{% endblock %}
</head>
<body>
  {% include "navbar.html" %}
  <main class="container my-4">
    {% block content %}{% endblock %}
  </main>
  {% include "footer.html" %}
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
  {% block scripts %}{% endblock %}
</body>
</html>
```

### `navbar.html`
```html
<nav class="navbar navbar-expand-lg navbar-light bg-white fixed-top shadow-sm">
  <div class="container">
    <a class="navbar-brand d-flex align-items-center" href="{% url 'inicio' %}">
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="bi bi-pc" viewBox="0 0 16 16">
        <path d="M5 1a2 2 0 0 0-2 2v7h10V3a2 2 0 0 0-2-2H5z"/>
        <path d="M0 12.5A1.5 1.5 0 0 1 1.5 11H14.5A1.5 1.5 0 0 1 16 12.5v.5a.5.5 0 0 1-.5.5H.5a.5.5 0 0 1-.5-.5v-.5z"/>
      </svg>
      <span class="ms-2">Sistema de Administración de Computadoras y Accesorios</span>
    </a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navmenu">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navmenu">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio' %}">Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" id="prodMenu" role="button" data-bs-toggle="dropdown">Productos</a>
          <ul class="dropdown-menu dropdown-menu-end" aria-labelledby="prodMenu">
            <li><a class="dropdown-item" href="{% url 'agregar_producto' %}">Agregar Producto</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_productos' %}">Ver Productos</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" id="provMenu" role="button" data-bs-toggle="dropdown">Proveedores</a>
          <ul class="dropdown-menu dropdown-menu-end" aria-labelledby="provMenu">
            <li><a class="dropdown-item" href="#">Agregar Proveedor</a></li>
            <li><a class="dropdown-item" href="#">Ver Proveedores</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" id="factMenu" role="button" data-bs-toggle="dropdown">Facturas</a>
          <ul class="dropdown-menu dropdown-menu-end" aria-labelledby="factMenu">
            <li><a class="dropdown-item" href="#">Agregar Factura</a></li>
            <li><a class="dropdown-item" href="#">Ver Facturas</a></li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</nav>
```

### `footer.html`
```html
<footer class="py-2">
  <div class="container d-flex justify-content-between small">
    <div>© Derechos reservados</div>
    <div id="fechaFooter"></div>
    <div>Creado por Ing. Eliseo Nava, Cbtis 128</div>
  </div>
  <script>
    document.getElementById('fechaFooter').innerText = new Date().toLocaleDateString();
  </script>
</footer>
```

### `inicio.html`
```html
{% extends "base.html" %}
{% block title %}Inicio - Sistema de Computadoras{% endblock %}
{% block content %}
  <div class="row">
    <div class="col-md-7">
      <h1>Bienvenido al Sistema de Administración</h1>
      <p>Fecha del sistema: {{ hoy }}</p>
      <p>Este sistema permite administrar Productos, Proveedores y Facturas.</p>
    </div>
    <div class="col-md-5">
      <img src="https://images.unsplash.com/photo-1518770660439-4636190af475?q=80&w=1000&auto=format&fit=crop&ixlib=rb-4.0.3&s=0" alt="Computadoras" class="img-fluid rounded shadow">
    </div>
  </div>
{% endblock %}
```

---

## 21. Crear subcarpeta `producto` dentro de `templates`
Ruta: `app_Computadoras/templates/producto/`

---

## 22. Archivos HTML para producto

### `agregar_producto.html`
```html
{% extends "base.html" %}
{% block content %}
<h2>Agregar Producto</h2>
<form method="post">
  {% csrf_token %}
  <div class="mb-3">
    <label class="form-label">ID (opcional)</label>
    <input class="form-control" name="id">
  </div>
  <div class="mb-3">
    <label class="form-label">Nombre</label>
    <input class="form-control" name="nombre" required>
  </div>
  <div class="mb-3">
    <label class="form-label">Descripción</label>
    <textarea class="form-control" name="descripcion"></textarea>
  </div>
  <div class="row">
    <div class="col">
      <label class="form-label">Precio</label>
      <input class="form-control" name="precio" type="number" step="0.01">
    </div>
    <div class="col">
      <label class="form-label">Stock</label>
      <input class="form-control" name="stock" type="number">
    </div>
    <div class="col">
      <label class="form-label">Categoría (ID)</label>
      <input class="form-control" name="categoria" type="number">
    </div>
  </div>
  <div class="mb-3 mt-3">
    <label class="form-label">Proveedor (ID)</label>
    <input class="form-control" name="proveedor" placeholder="ID del proveedor (opcional)">
  </div>
  <button class="btn btn-primary">Guardar</button>
  <a href="{% url 'ver_productos' %}" class="btn btn-secondary">Volver</a>
</form>
{% endblock %}
```

### `ver_productos.html`
```html
{% extends "base.html" %}
{% block content %}
<h2>Productos</h2>
<a class="btn btn-success mb-3" href="{% url 'agregar_producto' %}">Agregar Producto</a>
<table class="table table-striped table-bordered">
  <thead class="table-light">
    <tr>
      <th>ID</th><th>Nombre</th><th>Precio</th><th>Stock</th><th>Proveedor</th><th>Acciones</th>
    </tr>
  </thead>
  <tbody>
    {% for p in productos %}
    <tr>
      <td>{{ p.id }}</td>
      <td>{{ p.nombre }}</td>
      <td>{{ p.precio }}</td>
      <td>{{ p.stock }}</td>
      <td>{% if p.proveedor %}{{ p.proveedor.nombre }} ({{ p.proveedor.id }}){% else %}-{% endif %}</td>
      <td>
        <a class="btn btn-sm btn-info" href="{% url 'actualizar_producto' p.id %}">Editar</a>
        <form method="post" action="{% url 'borrar_producto' p.id %}" style="display:inline;">
          {% csrf_token %}
          <button class="btn btn-sm btn-danger" type="submit">Borrar</button>
        </form>
      </td>
    </tr>
    {% empty %}
    <tr><td colspan="6" class="text-center">No hay productos.</td></tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}
```

### `actualizar_producto.html`
```html
{% extends "base.html" %}
{% block content %}
<h2>Actualizar Producto</h2>
<form method="post" action="{% url 'realizar_actualizacion_producto' producto.id %}">
  {% csrf_token %}
  <div class="mb-3">
    <label class="form-label">Nombre</label>
    <input class="form-control" name="nombre" value="{{ producto.nombre }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Descripción</label>
    <textarea class="form-control" name="descripcion">{{ producto.descripcion }}</textarea>
  </div>
  <div class="row">
    <div class="col">
      <label class="form-label">Precio</label>
      <input class="form-control" name="precio" value="{{ producto.precio }}">
    </div>
    <div class="col">
      <label class="form-label">Stock</label>
      <input class="form-control" name="stock" value="{{ producto.stock }}">
    </div>
    <div class="col">
      <label class="form-label">Categoría (ID)</label>
      <input class="form-control" name="categoria" value="{{ producto.categoria }}">
    </div>
  </div>
  <div class="mb-3 mt-3">
    <label class="form-label">Proveedor (ID)</label>
    <input class="form-control" name="proveedor" value="{% if producto.proveedor %}{{ producto.proveedor.id }}{% endif %}">
  </div>
  <button class="btn btn-primary">Guardar cambios</button>
  <a class="btn btn-secondary" href="{% url 'ver_productos' %}">Cancelar</a>
</form>
{% endblock %}
```

### `borrar_producto.html`
```html
{% extends "base.html" %}
{% block content %}
<h2>Confirmar borrado</h2>
<p>¿Deseas borrar el producto <strong>{{ producto.nombre }}</strong> (ID: {{ producto.id }})?</p>
<form method="post">
  {% csrf_token %}
  <button class="btn btn-danger">Sí, borrar</button>
  <a class="btn btn-secondary" href="{% url 'ver_productos' %}">Cancelar</a>
</form>
{% endblock %}
```

---

## 23. No utilizar `forms.py`
Se usan formularios HTML nativos y `request.POST` en vistas.

---

## 24. urls.py en `app_Computadoras` (crear `app_Computadoras/urls.py`)
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_computadoras, name='inicio'),
    path('productos/agregar/', views.agregar_producto, name='agregar_producto'),
    path('productos/', views.ver_productos, name='ver_productos'),
    path('productos/actualizar/<int:producto_id>/', views.actualizar_producto, name='actualizar_producto'),
    path('productos/actualizar/realizar/<int:producto_id>/', views.realizar_actualizacion_producto, name='realizar_actualizacion_producto'),
    path('productos/borrar/<int:producto_id>/', views.borrar_producto, name='borrar_producto'),
]
```

---

## 25. Agregar `app_Computadoras` en `backend_Computadoras/settings.py`
Agregar en `INSTALLED_APPS`:
```python
INSTALLED_APPS = [
    # ... apps de Django ...
    'app_Computadoras',
]
```

---

## 26. Configurar `backend_Computadoras/urls.py` para enlazar la app
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Computadoras.urls')),
]
```

---

## 27. Registrar modelos en `app_Computadoras/admin.py`
```python
from django.contrib import admin
from .models import Producto, Proveedor, Factura

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ('id','nombre','precio','stock','categoria','proveedor')
    search_fields = ('nombre',)

@admin.register(Proveedor)
class ProveedorAdmin(admin.ModelAdmin):
    list_display = ('id','nombre','pais')

@admin.register(Factura)
class FacturaAdmin(admin.ModelAdmin):
    list_display = ('id','fecha','total','metodo_pago')
```

Luego:
```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 27 (bis). Crear superusuario (opcional)
```bash
python manage.py createsuperuser
```

---

## 28. Diseño: colores suaves y modernos
Se usó Bootstrap y paleta clara en `base.html`: fondo `#f7fafc`, navbar blanca, sombras suaves.

---

## 29. Crear la estructura completa de carpetas y archivos
Ya definida al inicio. Crear archivos en VS Code y copiar los contenidos proporcionados.

---

## 30. Proyecto totalmente funcional (nota)
Con las migraciones realizadas y servidor en el puerto 8018, el CRUD de Productos funcionará: `Agregar`, `Ver`, `Editar`, `Borrar`.

---

## 31. Ejecutar servidor en el puerto 8018
```bash
python manage.py runserver 8018
```

---

## Archivos extra (helpers)

### `.gitignore` sugerido
```
# Python
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.venv/
# VS Code
.vscode/
# SQLite
db.sqlite3
```

---

### `README` rápido (opcional)
```markdown
# UIII_Computadoras_1291
Proyecto Django para administrar Computadoras y Accesorios.
1. Crear entorno virtual: python -m venv .venv
2. Activar: source .venv/bin/activate
3. Instalar dependencias: pip install django
4. Crear app: python manage.py startapp app_Computadoras
5. Añadir app a settings.py
6. Migraciones: python manage.py makemigrations && python manage.py migrate
7. Ejecutar: python manage.py runserver 8018
```

---

## Fin del archivo
