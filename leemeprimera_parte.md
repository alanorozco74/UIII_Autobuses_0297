# Sistema de Gestión de Autobuses - Código Generado

## 🚀 Instrucciones de Instalación

### 1. Crear entorno virtual
```bash
python -m venv .venv
2. Activar entorno virtual
Windows:

bash
.venv\Scripts\activate
Linux/Mac:

bash
source .venv/bin/activate
3. Instalar Django
bash
pip install django
4. Crear proyecto y aplicación
bash
django-admin startproject backend_Autobuses .
python manage.py startapp app_Autobuses
📁 CÓDIGO COMPLETO
models.py
python
from django.db import models

class Autobus(models.Model):
    TIPOS_SERVICIO = [
        ('Ejecutivo', 'Ejecutivo'),
        ('Económico', 'Económico'),
        ('Lujo', 'Lujo'),
        ('Primera Clase', 'Primera Clase'),
    ]
    
    ESTADOS = [
        ('Activo', 'Activo'),
        ('Mantenimiento', 'Mantenimiento'),
        ('Inactivo', 'Inactivo'),
    ]
    
    placa = models.CharField(max_length=10, unique=True)
    marca = models.CharField(max_length=50)
    modelo = models.CharField(max_length=50)
    capacidad_pasajeros = models.IntegerField()
    tipo_servicio = models.CharField(max_length=20, choices=TIPOS_SERVICIO)
    año_fabricacion = models.IntegerField()
    estado = models.CharField(max_length=20, choices=ESTADOS, default='Activo')
    
    def __str__(self):
        return f"{self.placa} - {self.marca} {self.modelo}"

class Ruta(models.Model):
    clave_ruta = models.CharField(max_length=10, unique=True)
    nombre_ruta = models.CharField(max_length=100)
    terminal_origen = models.CharField(max_length=100)
    terminal_destino = models.CharField(max_length=100)
    distancia_km = models.DecimalField(max_digits=8, decimal_places=2)
    duracion_estimada_min = models.IntegerField()
    precio_base = models.DecimalField(max_digits=8, decimal_places=2)
    autobus = models.ForeignKey(Autobus, on_delete=models.CASCADE, related_name="rutas")
    
    def __str__(self):
        return f"{self.clave_ruta} - {self.terminal_origen} a {self.terminal_destino}"

class Empleado(models.Model):
    PUESTOS = [
        ('Conductor', 'Conductor'),
        ('Ayudante', 'Ayudante'),
        ('Jefe de Terminal', 'Jefe de Terminal'),
        ('Vendedor de Boletos', 'Vendedor de Boletos'),
        ('Administrativo', 'Administrativo'),
        ('Mantenimiento', 'Mantenimiento'),
    ]
    
    nombre = models.CharField(max_length=50)
    apellido_paterno = models.CharField(max_length=50)
    apellido_materno = models.CharField(max_length=50)
    rfc = models.CharField(max_length=13, unique=True)
    puesto = models.CharField(max_length=30, choices=PUESTOS)
    fecha_contratacion = models.DateField()
    telefono = models.CharField(max_length=15)
    rutas = models.ManyToManyField(Ruta, related_name="empleados")
    
    def __str__(self):
        return f"{self.nombre} {self.apellido_paterno} {self.apellido_materno}"
views.py
python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Autobus

def inicio_autobuses(request):
    return render(request, 'inicio.html')

def agregar_autobus(request):
    if request.method == 'POST':
        placa = request.POST['placa']
        marca = request.POST['marca']
        modelo = request.POST['modelo']
        capacidad_pasajeros = request.POST['capacidad_pasajeros']
        tipo_servicio = request.POST['tipo_servicio']
        año_fabricacion = request.POST['año_fabricacion']
        estado = request.POST['estado']
        
        Autobus.objects.create(
            placa=placa,
            marca=marca,
            modelo=modelo,
            capacidad_pasajeros=capacidad_pasajeros,
            tipo_servicio=tipo_servicio,
            año_fabricacion=año_fabricacion,
            estado=estado
        )
        return redirect('ver_autobuses')
    
    return render(request, 'autobus/agregar_autobus.html')

def ver_autobuses(request):
    autobuses = Autobus.objects.all()
    return render(request, 'autobus/ver_autobuses.html', {'autobuses': autobuses})

def actualizar_autobus(request, autobus_id):
    autobus = get_object_or_404(Autobus, id=autobus_id)
    
    if request.method == 'POST':
        autobus.placa = request.POST['placa']
        autobus.marca = request.POST['marca']
        autobus.modelo = request.POST['modelo']
        autobus.capacidad_pasajeros = request.POST['capacidad_pasajeros']
        autobus.tipo_servicio = request.POST['tipo_servicio']
        autobus.año_fabricacion = request.POST['año_fabricacion']
        autobus.estado = request.POST['estado']
        autobus.save()
        return redirect('ver_autobuses')
    
    return render(request, 'autobus/actualizar_autobus.html', {'autobus': autobus})

def borrar_autobus(request, autobus_id):
    autobus = get_object_or_404(Autobus, id=autobus_id)
    
    if request.method == 'POST':
        autobus.delete()
        return redirect('ver_autobuses')
    
    return render(request, 'autobus/borrar_autobus.html', {'autobus': autobus})
urls.py (app_Autobuses)
python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_autobuses, name='inicio_autobuses'),
    path('agregar-autobus/', views.agregar_autobus, name='agregar_autobus'),
    path('ver-autobuses/', views.ver_autobuses, name='ver_autobuses'),
    path('actualizar-autobus/<int:autobus_id>/', views.actualizar_autobus, name='actualizar_autobus'),
    path('borrar-autobus/<int:autobus_id>/', views.borrar_autobus, name='borrar_autobus'),
]
admin.py
python
from django.contrib import admin
from .models import Autobus, Ruta, Empleado

@admin.register(Autobus)
class AutobusAdmin(admin.ModelAdmin):
    list_display = ['placa', 'marca', 'modelo', 'tipo_servicio', 'estado']
    list_filter = ['tipo_servicio', 'estado']
    search_fields = ['placa', 'marca']

@admin.register(Ruta)
class RutaAdmin(admin.ModelAdmin):
    list_display = ['clave_ruta', 'terminal_origen', 'terminal_destino', 'autobus']
    list_filter = ['autobus']
    search_fields = ['clave_ruta', 'terminal_origen']

@admin.register(Empleado)
class EmpleadoAdmin(admin.ModelAdmin):
    list_display = ['nombre', 'apellido_paterno', 'puesto', 'fecha_contratacion']
    list_filter = ['puesto']
    search_fields = ['nombre', 'rfc']
🎯 Comandos de Ejecución
bash
# Realizar migraciones
python manage.py makemigrations
python manage.py migrate

# Crear superusuario (opcional)
python manage.py createsuperuser

# Ejecutar servidor
python manage.py runserver 8036
Nota: Los templates HTML son extensos y se incluirán en la siguiente fase de desarrollo.

Código generado para el Sistema de Gestión de Autobuses - Primera Parte

text

4. **Haz clic en "Commit changes"**

## 🚀 **PASO 3: SUBIR PDF**

1. **Haz clic en "Add file" → "Upload files"**
2. **Arrastra tu PDF** (el que creaste en Google Docs)
3. **Cambia el nombre** a: `documento_requerimientos.pdf`
4. **Haz clic en "Commit changes"**

## 🚀 **PASO 4: SUBIR IMAGEN DB DESIGNER**

1. **Toma captura** de tu diseño en dbdesigner
2. **Guárdala** como `dbdesigner.png`
3. **En GitHub:** "Add file" → "Upload files"
4. **Arrastra la imagen** `dbdesigner.png`
5. **Haz clic en "Commit changes"**

## 🚀 **PASO 5: ENTREGAR EN CLASSROOM**

1. **Copia el link** de tu repositorio (ej: `https://github.com/tuusuario/UIII_Autobuses_0777`)
2. **Ve a Classroom** y pega el link
3. **¡Listo!**

## 📋 **RESUMEN DE ARCHIVOS QUE DEBES TENER:**

- ✅ `README.md`
- ✅ `leeme_primera_parte.md` 
- ✅ `documento_requerimientos.pdf`
- ✅ `dbdesigner.png`

**¿Listo para hacerlo paso por paso?** ¡Empieza con el PASO 1!
