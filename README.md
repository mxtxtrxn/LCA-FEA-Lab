# LCA-FEA-Lab

Motor propio de elementos finitos no lineal, en C++17, para simulación biomecánica **sintética** del ligamento cruzado anterior (LCA). No es una herramienta clínica y no usa datos de pacientes: la geometría, la orientación de las fibras de colágeno, los ángulos de inserción y la pretensión se generan de forma procedural con semillas fijas.

Estado actual: documentación inicial. **Todavía no hay código implementado** y el proyecto sigue pendiente de su propia aprobación.
-
## Objetivo

Implementar desde cero la cinemática, las leyes constitutivas, la cuadratura, el ensamblaje y el solucionador no lineal necesarios para simular el LCA bajo un escenario tipo Lachman o cajón anterior. La meta de largo plazo es sustituir corridas repetidas de FEA por un modelo sustituto pequeño entrenado solo con datos sintéticos, midiendo latencia en CPU sin hacer promesas de uso clínico real.

## Alcance previsto por fases

- **Baseline y contrato de datos:** verificar el toolchain offline, decidir el formato CSV/JSON de salida y confirmar que no entren datos reales de salud al repositorio.
- **Fase 1 — punto material:** cinemática de Lagrange total (`F`, `C`, `J`, `I1`, `I4`) y leyes hiperelásticas HGO / Veronda-Westmann con refuerzo exponencial de fibras, con verificación de la tangente por diferencias finitas.
- **Fase 2 — elemento y solver:** funciones de forma isoparamétricas Hex8/Tet4, cuadratura de Gauss, ensamblaje disperso, Newton-Raphson, condiciones de Dirichlet y Neumann, *patch test* y benchmarks de tracción uniaxial y corte simple.
- **Fase 3 — ligamento sintético:** malla procedural del LCA, campo sintético de fibras y escenario de desplazamiento tipo Lachman.
- **Fase 4 — aceleración y sustituto:** paralelización del ensamblaje con OpenMP si el compilador lo soporta, y modelo POD/SVD + MLP pequeño sobre el banco sintético, con medida de latencia en CPU.
- **Cierre:** robustez, documentación, capturas y notas para el revisor.

## Arquitectura modular prevista

La implementación se hará módulo por módulo, sin generar todo el proyecto de una vez:

- `src/kinematics.hpp` — tensores `F`, `C`, invariantes y operaciones tensoriales.
- `src/constitutive.hpp` — energía `Psi`, esfuerzo `S`, tangente `C` y esfuerzo de Cauchy.
- `src/elements.hpp` — funciones de forma, jacobianos y puntos de Gauss.
- `src/assembler.hpp` — ensamblaje de `K_T` y `f_int`.
- `src/solver.hpp` — Newton-Raphson incremental y condiciones de frontera.
- `src/surrogate_ai.py` — reducción POD y modelo sustituto pequeño.
- `tests/test_unit_cube.cpp` — validación física y convergencia.
- `dashboard/` — visor web estático o servido con la librería estándar de Python.

Estas rutas son una guía de diseño; **aún no existen como archivos**.

## Stack previsto

- C++17, CMake y CTest, compilado con gcc o clang en el escritorio Linux.
- Eigen y doctest se vendorizarán bajo `external/` cuando se implemente el núcleo, para no depender de `apt-get` ni de internet.
- Python 3 de la librería estándar para el servidor local del visor.
- HTML, CSS y JavaScript plano en Chrome, sin CDN, frameworks ni base de datos.
- NumPy y scikit-learn solo si ya están disponibles offline, para el modelo sustituto pequeño.
- Quedan fuera de alcance FEBio, Abaqus, Ansys, FEniCS, GPU, CUDA, JAX, PyTorch, MeshGraphNet, PINN, Docker, ParaView y toolkits nativos de interfaz.

## Datos, unidades y límites

- Unidades consistentes: mm, N y MPa.
- Solo datos analíticos o sintéticos con semillas fijas.
- No se usarán imágenes clínicas, geometrías de pacientes, historiales médicos ni datos personales, de menores, empleador o clientes.
- No se compartirán credenciales, tokens ni contraseñas.
- La interfaz será una app web abierta en Chrome; el escritorio no tiene internet, GPU, Docker, sonido ni gestores de paquetes del sistema.

## Aprobación

Este proyecto es independiente de los otros trabajos y necesita su **propia aprobación**, su propio reporte de uso de IA y su propio workspace antes de iniciar cualquier sesión. Antes de cada sesión: **New** en Labelbox, luego Hubstaff, y recién después abrir el workspace. Todo el texto que se envíe a Labelbox debe ser reescrito por el autor con sus palabras.

## Licencia

MIT. Ver `LICENSE`. Los componentes de terceros que se vendoricen conservarán sus propias licencias y se anotarán aquí y en `LICENSE` cuando se añadan.
