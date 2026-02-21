# Algoritmo de Shor: Implementación en Qiskit para N=15
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1lfGnrUHiqc8XUsmZahFQjEVrckLr4kpD?usp=sharing)

<p align="center">
  <img src="https://img.shields.io/badge/Qiskit-2.x-blue?logo=qiskit" alt="Qiskit">
  <img src="https://img.shields.io/badge/Python-3.10+-green?logo=python" alt="Python">
  <img src="https://img.shields.io/badge/License-MIT-yellow" alt="License">
  <img src="https://img.shields.io/badge/Universidad-ULL-red" alt="ULL">
</p>

**Autor:** Hugo Tapia  
**Institución:** Universidad de La Laguna (ULL)  
**Asignatura:** Microcredencial en Criptografía e Información Cuántica  
**Docentes:** Jose D. Escánez Expósito / Jorge F. García Díaz

**Fecha:** Febrero 2026

---

## 📋 Índice

1. [Introducción](#1-introducción)
2. [Fundamentos Teóricos](#2-fundamentos-teóricos)
3. [Estructura del Algoritmo](#3-estructura-del-algoritmo)
4. [Implementación](#4-implementación)
5. [Ejecución y Resultados](#5-ejecución-y-resultados)
6. [Estructura del Repositorio](#6-estructura-del-repositorio)
7. [Instalación y Uso](#7-instalación-y-uso)
8. [Referencias](#8-referencias)

---

## 1. Introducción

### 1.1 Contexto

El **Algoritmo de Shor**, publicado por Peter Shor en 1994, representa uno de los avances más significativos en computación cuántica. Este algoritmo demuestra que un ordenador cuántico puede factorizar números enteros en **tiempo polinómico** O(n³), mientras que los mejores algoritmos clásicos conocidos requieren **tiempo subexponencial** O(exp(n^(1/3))).

### 1.2 Implicaciones para la Criptografía

La importancia del algoritmo radica en sus implicaciones para la seguridad de sistemas criptográficos como **RSA**, cuya seguridad se fundamenta en la dificultad computacional de factorizar números grandes (típicamente de 2048 bits o más).

| Algoritmo | Complejidad | Tipo |
|-----------|-------------|------|
| Fuerza bruta | O(√N) | Clásico |
| Criba del cuerpo de números | O(exp(n^(1/3))) | Clásico |
| **Algoritmo de Shor** | **O(n³)** | **Cuántico** |

### 1.3 Objetivo del Trabajo

Implementar el Algoritmo de Shor utilizando **Qiskit** para factorizar **N = 15**, demostrando:

- La construcción del circuito cuántico
- La aplicación de Quantum Phase Estimation (QPE)
- El post-procesamiento clásico mediante fracciones continuas
- La obtención de los factores: **15 = 3 × 5**

---

## 2. Fundamentos Teóricos

### 2.1 El Problema de Factorización

Dado un número compuesto N, encontrar sus factores primos p y q tales que:

```
N = p × q
```

Para N = 15: **p = 3** y **q = 5**

### 2.2 Reducción a Búsqueda de Período

Shor transforma el problema de factorización en un problema de **búsqueda de período**:

> **Teorema:** Si encontramos el período `r` de la función f(x) = aˣ mod N (donde `a` es coprimo con N), entonces los factores de N pueden calcularse como:
>
> - **Factor 1:** gcd(a^(r/2) - 1, N)
> - **Factor 2:** gcd(a^(r/2) + 1, N)

### 2.3 Ejemplo con a=7, N=15

```
7¹ mod 15 = 7
7² mod 15 = 4
7³ mod 15 = 13
7⁴ mod 15 = 1  ← ¡El período es r = 4!
```

Aplicando el teorema:
```
x = 7^(4/2) mod 15 = 7² mod 15 = 4

gcd(4 - 1, 15) = gcd(3, 15) = 3  ✓
gcd(4 + 1, 15) = gcd(5, 15) = 5  ✓

Por lo tanto: 15 = 3 × 5
```

### 2.4 Quantum Phase Estimation (QPE)

El algoritmo cuántico encuentra el período mediante QPE:

1. **Superposición:** Crear superposición uniforme en el registro de control
2. **Operaciones controladas:** Aplicar U^(2^k) controlado por cada qubit
3. **QFT inversa:** Transformar la información de fase en valores medibles
4. **Medición:** Obtener s/r donde r es el período buscado

---

## 3. Estructura del Algoritmo

### 3.1 Diagrama de Flujo General

```
┌─────────────────────────────────────────────────────────────────┐
│                    ALGORITMO DE SHOR                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────── ┐   │
│  │              PRE-PROCESAMIENTO (Clásico)                 │   │
│  │  • ¿N es par? → Factor trivial: 2                        │   │
│  │  • ¿N es potencia de primo? → Resolver directamente      │   │
│  │  • Elegir 'a' aleatorio (2 < a < N-1)                    │   │
│  │  • ¿gcd(a,N) > 1? → Factor encontrado sin Shor           │   │
│  └─────────────────────────┬─────────────────────────────── ┘   │
│                            │ gcd(a,N) = 1                       │
│                            ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              FASE I: CUÁNTICA                           │    │
│  │  • Crear circuito con m qubits de control + n auxiliares│    │
│  │  • Aplicar Hadamard al registro de control              │    │
│  │  • Aplicar U^(2^k) controlado (exponenciación modular)  │    │
│  │  • Aplicar QFT inversa                                  │    │
│  │  • Medir registro de control → obtener 'y'              │    │
│  └─────────────────────────┬───────────────────────────────┘    │
│                            │                                    │
│                            ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              FASE II: CLÁSICA                           │    │
│  │  • Aplicar fracciones continuas a y/2^m                 │    │
│  │  • Obtener candidato r                                  │    │
│  │  • ¿r es par? (si no, reintentar)                       │    │
│  │  • ¿a^(r/2) ≡ -1 mod N? (si sí, reintentar)             │    │
│  │  • Calcular gcd(a^(r/2) ± 1, N)                         │    │
│  └─────────────────────────┬───────────────────────────────┘    │
│                            │                                    │
│                            ▼                                    │
│                   ┌─────────────────┐                           │
│                   │ FACTORES DE N   │                           │
│                   │   15 = 3 × 5    │                           │
│                   └─────────────────┘                           │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Circuito Cuántico

```
         ┌───┐                                         ┌───────────┐┌─┐
control₀:┤ H ├────●────────────────────────────────────┤           ├┤M├
         ├───┤    │                                    │           │└╥┘
control₁:┤ H ├────┼──────●─────────────────────────────┤           ├─╫─
         ├───┤    │      │                             │   QFT⁻¹   │ ║
control₂:┤ H ├────┼──────┼──────●──────────────────────┤           ├─╫─
         ├───┤    │      │      │                      │           │ ║
   ...   │...│   ...    ...    ...                     │           │...
         ├───┤    │      │      │           │          └───────────┘ ║
control₇:┤ H ├────┼──────┼──────┼───────────●────────────────────────╫─
         └───┘    │      │      │           │                        ║
              ┌───┴───┐┌─┴──┐┌──┴──┐    ┌───┴───┐                    ║
 target₀: ─X──┤       ││    ││     │    │       │                    ║
              │  U¹   ││ U² ││ U⁴  │    │ U¹²⁸  │                    ║
 target₁: ────┤ mod N ││mod ││mod  │....│ mod N │                    ║
              │       ││ N  ││ N   │    │       │                    ║
 target₂: ────┤       ││    ││     │    │       │                    ║
              │       ││    ││     │    │       │                    ║
 target₃: ────┤       ││    ││     │    │       │                    ║
              └───────┘└────┘└─────┘    └───────┘                    ║
                                                                     ║
medición: ═══════════════════════════════════════════════════════════╩═
```

### 3.3 Parámetros del Circuito

| Parámetro | Valor para N=15 | Descripción |
|-----------|-----------------|-------------|
| n | 4 | Qubits para representar N (2⁴ = 16 > 15) |
| m | 8 | Qubits de control (2n para precisión) |
| Total qubits | 12 | m + n = 8 + 4 |

---



## 4. Implementación
### 4.1 Pre-procesamiento



```python
def pre_procesamiento_shor(N):
    # 1. ¿N es par?
    if N % 2 == 0:
        return f"Factor trivial: 2"
    
    # 2. ¿N es potencia de primo?
    if es_potencia_de_primo(N):
        return f"N = p^k, resolver directamente"
    
    # 3. Elegir 'a' aleatorio
    a = random.randint(3, N - 2)
    
    # 4. ¿gcd(a, N) > 1?
    d = math.gcd(a, N)
    if d > 1:
        return f"Factor encontrado: {d}"
    
    # 5. Pasar a fase cuántica
    return f"Ejecutar Shor con a={a}"
```



### 4.2 Puerta de Exponenciación Modular

```python
def amodN(a, power, N):
    """
    Crea la puerta unitaria: U|x⟩ = |(a^power · x) mod N⟩
    """
    n = N.bit_length()
    u_matrix = np.zeros((2**n, 2**n))
    
    for x in range(2**n):
        if x < N:
            # Multiplicación modular
            u_matrix[(x * pow(a, power, N)) % N, x] = 1
        else:
            # Estados fuera de rango: identidad
            u_matrix[x, x] = 1
    
    return UnitaryGate(u_matrix, label=f"{a}^{power} mod {N}")
```

### 4.3 Circuito de Shor

```python
def crear_circuito_shor(a, N):
    n = N.bit_length()  # 4 para N=15
    m = 2 * n           # 8 qubits de control
    
    # Registros
    reg_control = QuantumRegister(m, 'control')
    reg_objetivo = QuantumRegister(n, 'target')
    bits_clasicos = ClassicalRegister(m, 'medicion')
    qc = QuantumCircuit(reg_control, reg_objetivo, bits_clasicos)
    
    # 1. Inicializar |1⟩ en registro objetivo
    qc.x(reg_objetivo[0])
    
    # 2. Superposición en control
    qc.h(reg_control)
    
    # 3. Exponenciación modular controlada
    for i in range(m):
        puerta_u = amodN(a, 2**i, N).control()
        qc.append(puerta_u, [reg_control[i]] + list(reg_objetivo))
    
    # 4. QFT inversa
    qc.append(QFT(m).inverse(), reg_control)
    
    # 5. Medición
    qc.measure(reg_control, bits_clasicos)
    
    return qc
```

### 4.4 Post-procesamiento con Fracciones Continuas

```python
def encontrar_r(y, m, a, N):
    """
    Extrae el período r usando fracciones continuas.
    """
    # Convertir medición a fase
    fase = y / (2**m)
    
    # Aproximar con fracción simple
    fraccion = Fraction(fase).limit_denominator(N)
    r_candidato = fraccion.denominator
    
    # Verificar: a^r mod N = 1
    if pow(a, r_candidato, N) == 1:
        return r_candidato
    return None
```

### 4.5 Clase Integrada

```python
class ShorAlgorithm:
    def __init__(self, N, max_attempts=7):
        self.N = N
        self.n = N.bit_length()
        self.m = 2 * self.n
        self.max_attempts = max_attempts
    
    def execute(self):
        # Pre-procesamiento
        if self._is_N_invalid():
            return None
        
        for intento in range(self.max_attempts):
            a = random.randint(3, self.N - 2)
            
            # ¿Suerte con GCD?
            if math.gcd(a, self.N) > 1:
                return (math.gcd(a, self.N), self.N // math.gcd(a, self.N))
            
            # Ejecutar circuito cuántico
            r = self._encontrar_orden_r(a)
            
            if r and r % 2 == 0:
                x = pow(a, r // 2, self.N)
                if x != self.N - 1:
                    f1 = math.gcd(x - 1, self.N)
                    if 1 < f1 < self.N:
                        return (f1, self.N // f1)
        
        return None
```

## 5. Ejecución y Resultados

### 5.1 Ejecución

```python
shor = ShorAlgorithm(15)
factores = shor.execute(draw=True)
```


### 5.2 Resultados de las Mediciones

Para a=7, N=15, las mediciones típicas son:

| Medición (binario) | Decimal | Fase | Fracción | r candidato |
|--------------------|---------|------|----------|-------------|
| 00000000 | 0 | 0.0000 | 0/1 | — |
| 01000000 | 64 | 0.2500 | 1/4 | **4** ✓ |
| 10000000 | 128 | 0.5000 | 1/2 | 2 |
| 11000000 | 192 | 0.7500 | 3/4 | **4** ✓ |

### 5.3 Cálculo de Factores

```
r = 4 (período encontrado)

x = a^(r/2) mod N = 7² mod 15 = 49 mod 15 = 4

gcd(x - 1, N) = gcd(3, 15) = 3  ← Factor 1
gcd(x + 1, N) = gcd(5, 15) = 5  ← Factor 2

✓ RESULTADO: 15 = 3 × 5
```

### 5.4 Salida del Programa

```
===== Intento 1/7 =====
[START] Base elegida a: 7
> a y N son coprimos. Iniciando búsqueda del orden...
  > Ejecución 1: y=64 => Candidato a r=4
  > Ejecución 2: y=192 => Candidato a r=4
  > Ejecución 3: y=128 => Candidato a r=2
  > Ejecución 4: y=64 => Candidato a r=4

Resumen de mediciones para a=7:
 Phase  Fraction  Guess for r
 0.2500      1/4            4
 0.7500      3/4            4
 0.5000      1/2            2
 0.2500      1/4            4

  >> LCM Final de candidatos: 4
[DONE] ¡Éxito! r=4. Factores encontrados: 3 * 5
```

---

## 6. Estructura del Repositorio

```
shor-algorithm-ull/
│
├── README.md                 # Este archivo
├── LICENSE                   # Licencia MIT
│
├── src/
│   └── shor_algorithm.py     # Implementación de la clase ShorAlgorithm
│
├── notebooks/
│   └── Algoritmo_de_Shor_N15.ipynb    # Notebook completo ejecutable
│
├── docs/
│   ├── teoria_shor.md                 # Fundamentos teóricos ampliados
│   └── diagramas/                     # Diagramas y figuras
│
└── images/
    ├── circuito_shor.png              # Imagen del circuito
    └── diagrama_flujo.png             # Diagrama de flujo del algoritmo
```

---

## 7. Instalación y Uso

### 7.1 Requisitos

- Python 3.10+
- Qiskit 2.x
- Qiskit Aer

### 7.2 Instalación

```bash
# Clonar repositorio
git clone https://github.com/tu-usuario/shor-algorithm-ull.git
cd shor-algorithm-ull

# Instalar dependencias
pip install qiskit qiskit-aer matplotlib numpy pandas
```

### 7.3 Ejecución en Google Colab

1. Abrir [Google Colab](https://colab.research.google.com)
2. Subir el notebook(https://colab.research.google.com/drive/1lfGnrUHiqc8XUsmZahFQjEVrckLr4kpD?usp=sharing)
3. Ejecutar todas las celdas

### 7.4 Ejecución Local

```python
from src.shor_algorithm import ShorAlgorithm

# Factorizar N=15
shor = ShorAlgorithm(15)
factores = shor.execute()
print(f"15 = {factores[0]} × {factores[1]}")
```

---

## 8. Referencias

1. **Shor, P. W.** (1994). "Algorithms for quantum computation: discrete logarithms and factoring". *Proceedings 35th Annual Symposium on Foundations of Computer Science*. IEEE.

2. **Nielsen, M. A., & Chuang, I. L.** (2010). *Quantum Computation and Quantum Information*. Cambridge University Press.

3. **Qiskit Documentation**. IBM Quantum. https://qiskit.org/documentation/

4. **Apuntes de la Microcredencial en Criptografía e Información Cuántica**. Universidad de La Laguna, 2026. Jose D. Escánez Expósito / Jorge F. García Díaz


---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Ver el archivo [LICENSE](LICENSE) para más detalles.

---

<p align="center">
  <b>Universidad de La Laguna</b><br>
  Microcredencial en Criptografía e Información Cuántica<br>
  2026
</p>
