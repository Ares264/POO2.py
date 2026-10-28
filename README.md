

# =====================================================================
# PARTE 1: EL PROBLEMA (HERENCIA MÚLTIPLE Y MRO)
# =====================================================================

class MatriculaGeneral:
    def costo(self):
        return 60.0

class MatriculaExtemporanea(MatriculaGeneral):
    def costo(self):
        return super().costo() + 15.0

class MatriculaDesaprobado(MatriculaGeneral):
    def costo(self):
        return super().costo() + 15.0

class MatriculaBeca(MatriculaGeneral):
    def costo(self):
        return super().costo() / 2.0


# Programador 1 (Luis): Beca, Desaprobado, Extemporánea
class Mat_Luis(MatriculaBeca, MatriculaDesaprobado, MatriculaExtemporanea):
    pass

# Programador 2 (Rosa): Desaprobado, Extemporánea, Beca
class Mat_Rosa(MatriculaDesaprobado, MatriculaExtemporanea, MatriculaBeca):
    pass

print("--- 1. ANÁLISIS DE LAS VERSIONES CON HERENCIA MÚLTIPLE ---")
luis = Mat_Luis()
rosa = Mat_Rosa()

print(f"Orden de ejecución de Luis: {[c.__name__ for c in Mat_Luis.__mro__]}")
print(f"Pago de Luis: S/ {luis.costo():.2f}") 
# Cálculo Luis: Base(60) -> Ext(+15) = 75 -> Desap(+15) = 90 -> Beca(/2) = 45.0

print(f"\nOrden de ejecución de Rosa: {[c.__name__ for c in Mat_Rosa.__mro__]}")
print(f"Pago de Rosa: S/ {rosa.costo():.2f}") 
# Cálculo Rosa: Base(60) -> Beca(/2) = 30 -> Ext(+15) = 45 -> Desap(+15) = 60.0

print("\n--- 2. PRUEBA DEL ERROR DE JERARQUÍA ---")
try:
    # Intentar poner la clase general antes que la especialización
    class ErrorMatricula(MatriculaGeneral, MatriculaBeca):
        pass
except TypeError as e:
    print(f"Resultado al poner la general primero: Error capturado -> {e}")


# =====================================================================
# PARTE 2: EL REDISEÑO (SOLUCIÓN BÁSICA Y EXPLÍCITA)
# =====================================================================
# Para evitar que el cálculo dependa de un detalle oculto (el MRO de la herencia múltiple),
# centralizamos los datos usando atributos básicos (Lab 02) y calculamos el costo
# con métodos explícitos donde la OTI decide el orden exacto.

class MatriculaEstudiante:
    def __init__(self, es_extemporanea=False, es_desaprobado=False, tiene_beca=False):
        self.monto_base = 60.0
        self.es_extemporanea = es_extemporanea
        self.es_desaprobado = es_desaprobado
        self.tiene_beca = tiene_beca

    def calcular_recargos_luego_beca(self):
        """Escenario A: Suma recargos y al final aplica la beca al monto total."""
        total = self.monto_base
        if self.es_extemporanea:
            total += 15.0
        if self.es_desaprobado:
            total += 15.0
        if self.tiene_beca:
            total = total / 2.0
        return total

    def calcular_beca_luego_recargos(self):
        """Escenario B: Aplica beca solo al monto base, luego suma los recargos fijos."""
        total = self.monto_base
        if self.tiene_beca:
            total = total / 2.0
        if self.es_extemporanea:
            total += 15.0
        if self.es_desaprobado:
            total += 15.0
        return total

print("\n--- 3. SOLUCIÓN MEDIANTE REDISEÑO BÁSICO ---")
# Creamos a los estudiantes con las 3 condiciones al mismo tiempo
estudiante = MatriculaEstudiante(es_extemporanea=True, es_desaprobado=True, tiene_beca=True)

print("Ahora la OTI puede calcular los montos controlando el orden matemático:")
print(f"Si se aplican los recargos y luego la beca: S/ {estudiante.calcular_recargos_luego_beca():.2f}")
print(f"Si se aplica la beca y luego los recargos: S/ {estudiante.calcular_beca_luego_recargos():.2f}")
