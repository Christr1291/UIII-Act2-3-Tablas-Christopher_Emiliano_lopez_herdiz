¡Claro! Aquí tienes todo el código listo para copiar y pegar, siguiendo los pasos que mencionaste:

### Estructura de directorios

1. `UIII_Computadoras_1291/`

   * `backend_Computadoras/`

     * `app_Computadoras/`

       * `migrations/`
       * `templates/`

         * `producto/`

           * `agregar_producto.html`
           * `ver_productos.html`
           * `actualizar_producto.html`
           * `borrar_producto.html`
         * `base.html`
         * `header.html`
         * `navbar.html`
         * `footer.html`
         * `inicio.html`
       * `models.py`
       * `views.py`
       * `urls.py`
       * `admin.py`
       * `apps.py`
     * `backend_Computadoras/`

       * `settings.py`
       * `urls.py`
       * `wsgi.py`
       * `asgi.py`

---

### 1. `models.py` (en `app_Computadoras`)

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
    categoria = models.IntegerField() # Asumiendo que categoria es un ID, si fuera una tabla, necesitaría un ForeignKey
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
    cliente_id = models.IntegerField() # Asumiendo que cliente_id es un ID, si fuera una tabla, necesitaría un ForeignKey
    empleado = models.CharField(max_length=255, blank=True, null=True)
    observaciones = models.TextField(blank=True, null=True)

    def __str__(self):
        return f"Factura #{self.id} - {self.fecha}"
```

---

### 2. `views.py` (en `app_Computadoras`)

```python
from django.shortcuts import render, redirect
from .models import Producto, Proveedor
from django.http import HttpResponse

# Función para la vista de inicio de productos
def inicio_computadoras(request):
    return render(request, 'inicio.html')

# Función para agregar un nuevo producto
def agregar_producto(request):
    if request.method == "POST":
        nombre = request.POST['nombre']
        descripcion = request.POST['descripcion']
        precio = request.POST['precio']
        stock = request.POST['stock']
        categoria = request.POST['categoria']
        proveedor_id = request.POST['proveedor']
        proveedor = Proveedor.objects.get(id=proveedor_id)
        Producto.objects.create(nombre=nombre, descripcion=descripcion, precio=precio, stock=stock, categoria=categoria, proveedor=proveedor)
        return redirect('ver_productos')
    return render(request, 'producto/agregar_producto.html')

# Función para ver productos
def ver_productos(request):
    productos = Producto.objects.all()
    return render(request, 'producto/ver_productos.html', {'productos': productos})

# Función para actualizar producto
def actualizar_producto(request, id):
    producto = Producto.objects.get(id=id)
    if request.method == "POST":
        producto.nombre = request.POST['nombre']
        producto.descripcion = request.POST['descripcion']
        producto.precio = request.POST['precio']
        producto.stock = request.POST['stock']
        producto.categoria = request.POST['categoria']
        producto.save()
        return redirect('ver_productos')
    return render(request, 'producto/actualizar_producto.html', {'producto': producto})

# Función para borrar un producto
def borrar_producto(request, id):
    producto = Producto.objects.get(id=id)
    producto.delete()
    return redirect('ver_productos')
```

---

### 3. `urls.py` (en `app_Computadoras`)

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_computadoras, name='inicio_computadoras'),
    path('agregar/', views.agregar_producto, name='agregar_producto'),
    path('ver/', views.ver_productos, name='ver_productos'),
    path('actualizar/<int:id>/', views.actualizar_producto, name='actualizar_producto'),
    path('borrar/<int:id>/', views.borrar_producto, name='borrar_producto'),
]
```

---

### 4. `base.html` (en `templates`)

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema de Administración de Computadoras</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}
    
    <div class="container mt-4">
        {% block content %}
        {% endblock %}
    </div>

    {% include 'footer.html' %}
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

### 5. `navbar.html` (en `templates`)

```html
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <a class="navbar-brand" href="#">Sistema de Administración de Computadoras y Accesorios</a>
  <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
    <span class="navbar-toggler-icon"></span>
  </button>
  <div class="collapse navbar-collapse" id="navbarNav">
    <ul class="navbar-nav">
      <li class="nav-item"><a class="nav-link" href="#">Inicio</a></li>
      <li class="nav-item dropdown">
        <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
          Productos
        </a>
        <ul class="dropdown-menu" aria-labelledby="navbarDropdown">
          <li><a class="dropdown-item" href="#">Agregar Producto</a></li>
          <li><a class="dropdown-item" href="#">Ver Productos</a></li>
          <li><a class="dropdown-item" href="#">Actualizar Producto</a></li>
          <li><a class="dropdown-item" href="#">Borrar Producto</a></li>
        </ul>
      </li>
      <li class="nav-item dropdown">
        <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
          Proveedores
        </a>
        <ul class="dropdown-menu" aria-labelledby="navbarDropdown">
          <li><a class="dropdown-item" href="#">Agregar Proveedor</a></li>
          <li><a class="dropdown-item" href="#">Ver Proveedores</a></li>
          <li><a class="dropdown-item" href="#">Actualizar Proveedor</a></li>
          <li><a class="dropdown-item" href="#">Borrar Proveedor</a></li>
        </ul>
      </li>
      <li class="nav-item dropdown">
        <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
          Facturas
        </a>
        <ul class="dropdown-menu" aria-labelledby="navbarDropdown">
          <li><a class="dropdown-item" href="#">Agregar Factura</a></li>
          <li><a class="dropdown-item" href="#">Ver Facturas</a></li>
          <li><a class="dropdown-item" href="#">Actualizar Factura</a></li>
          <li><a class="dropdown-item" href="#">Borrar Factura</a></li>
        </ul>
      </li>
    </ul>
  </div>
</nav>
```

---

### 6. `footer.html` (en `templates`)

```html
<footer class="fixed-bottom bg-light py-2 text-center">
  <p>&copy; {{ current
```


_year }} - Todos los derechos reservados. <br>Creado por Ing. Eliseo Nava, Cbtis 128</p>

</footer>
```

---

### 7. `inicio.html` (en `templates`)

```html
{% extends 'base.html' %}

{% block content %}
  <h1>Bienvenido al Sistema de Administración de Computadoras y Accesorios</h1>
  <p>Aquí podrás gestionar productos, proveedores y facturas de tu negocio.</p>
  <img src="https://via.placeholder.com/600x400?text=Computadoras+y+Accesorios" alt="Computadoras" class="img-fluid">
{% endblock %}
```

---

### 8. Configuración en `settings.py` (en `backend_Computadoras`)

Agrega `app_Computadoras` a la lista de `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # Otros apps...
    'app_Computadoras',
]
```

---

### 9. Configuración en `urls.py` (en `backend_Computadoras`)

Asegúrate de incluir las URLs de `app_Computadoras` en el archivo principal `urls.py` de tu proyecto:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Computadoras.urls')),  # Incluyendo las URLs de la app_Computadoras
]
```

---

¡Listo! Ahora puedes copiar y pegar este código en los archivos correspondientes dentro de tu proyecto.
