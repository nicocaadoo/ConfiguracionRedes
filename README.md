# Routing Protocols — Prácticas en Cisco Packet Tracer

Repositorio con las prácticas de enrutamiento realizadas para la materia **Integración de seguridad informática en redes y sistemas de software (Gpo 421)** del **Tecnológico de Monterrey**.

Las prácticas recorren, en orden, los principales mecanismos de enrutamiento: rutas estáticas, RIP, EIGRP y OSPF multiárea con redistribución hacia EIGRP.

- **Profesora:** Edith Carolina Arias Serna
- **Herramienta:** Cisco Packet Tracer (routers ISR4331, switches 2960-24TT)

## Equipo

| Integrante | Matrícula |
|---|---|
| Nicolas Casillas Larrañaga | A01787292 |
| Paulina Cortez Balvanera | A01782041 |
| María Espínola Forcén | A01787172 |

## Índice de prácticas

| # | Práctica | Tema | Entrega |
|---|---|---|---|
| 1.4 | [Static Routing](./1.4-static-routing/) | Rutas estáticas, default routes y rutas estáticas flotantes | 19/09/2026 |
| 1.6.1 | [RIPv1 y RIPv2](./1.6.1-rip/) | Distancia vector, métrica por saltos, VLSM | Septiembre 2026 |
| 1.7.1 | [EIGRP](./1.7.1-eigrp/) | EIGRP AS 100, métricas por ancho de banda y retardo | Septiembre 2026 |
| 1.8.1 | [OSPF](./1.8.1-ospf/) | OSPF multiárea, LSAs, default route y redistribución con EIGRP | 18/10/2026 |

## Estructura del repositorio

```
.
├── README.md
├── EIGRP/
│   ├── Documentation/
│   |   └── 1.7.1_ACTIVIDAD_EIGRP.pdf  
│   └── PacketTracer/
│       └── EIGRP.pkt
├── OSPF/
│   ├── Documentation/
|   |   └── 1.8.1_Actividad_OSPF.pdf
│   └── PacketTracer/
│       └── OSPF.pkt
├── RIPV1_RIPV2/
│   ├── Documentation/
│   |   └── 1.6.1_ACTIVIDAD_RIPv1_y_RIPv2.pdf
│   └── PacketTracer/
│       └── RIPV1V2.pkt
└── StaticRouting/
    ├── Documentation/
    |   └── Static-Routing.pdf
    └── PacketTracer/
        └── StaticRouting.pkt

```


---

## 1.4 — Static Routing

**Objetivo:** configurar y verificar rutas estáticas, rutas por defecto y rutas estáticas flotantes en una red de seis routers que representan zonas de México: Los Cabos, Guadalajara (GDL), México (MEX), Monterrey (MTY), Tlaxcala y un router de Internet (R_INET).

**Desarrollo**

1. **Static routes:** se armó la topología y se configuraron rutas estáticas con sus máscaras en cada router. Se verificó con comandos de consulta y `ping` entre PCs.
2. **Default routes:** se eliminaron las rutas estáticas de Los Cabos y Tlaxcala y se reemplazaron por una ruta por defecto hacia su único vecino. Se agregó R_INET conectado a Monterrey y se configuraron las rutas por defecto para salir a Internet.
3. **Floating static routes:** se agregó una segunda red/servidor de Internet para tener dos caminos de salida. Se configuró una ruta con distancia administrativa mayor (200) como respaldo y se apagaron interfaces para simular una falla y comprobar el cambio inmediato de ruta.

**Ejemplo de configuración**

```
! Los Cabos y Tlaxcala: una sola salida
ip route 0.0.0.0 0.0.0.0 192.168.53.53      ! Los Cabos
ip route 0.0.0.0 0.0.0.0 192.168.14.13      ! Tlaxcala

! GDL: ruta principal y flotante
ip route 0.0.0.0 0.0.0.0 172.16.23.125
ip route 0.0.0.0 0.0.0.0 172.16.13.33 200
```

---

## 1.6.1 — RIPv1 y RIPv2

**Objetivo:** analizar el funcionamiento de RIP: distancia administrativa, métrica, rutas disponibles y adaptación ante la caída de enlaces, verificando con `tracert`.

**Topología:** MTY, GDL_1, GDL_2, MEX y QRO.

**Resultados principales**

| Concepto | Resultado |
|---|---|
| Distancia administrativa de RIP | 120 |
| Rutas 50.50.50.0/24 → 40.40.40.0/24 | 2 (vía 60.60.60.66 y vía 172.16.13.33) |
| Métrica inicial | 2 saltos |
| Al desactivar GDL_1–GDL_2 | Queda 1 ruta vía 172.16.13.33 |
| Al desactivar también GDL_1–MEX | Nueva ruta vía 172.16.23.125 con 3 saltos |
| Convergencia de RIPv2 | ~30 segundos |

**RIPv1 vs RIPv2:** RIPv2 incluye la máscara de subred en sus actualizaciones, por lo que soporta VLSM y permite desactivar la sumarización automática. Ambas versiones usan distancia administrativa 120 y saltos como métrica.

```
conf t
router rip
 version 2
 no auto-summary
```

---

## 1.7.1 — EIGRP

**Objetivo:** configurar EIGRP 100, identificar rutas preferidas y métricas, y observar rutas alternativas al desactivar enlaces. Usa la misma topología que la práctica de RIP.

**Resultados principales** (ruta de 50.50.50.0/24 hacia 40.40.40.0/24, distancia administrativa 90)

| Escenario | Vía | Métrica |
|---|---|---|
| Red completa | 60.60.60.66 (Gi0/0/1) | 2,170,368 |
| Sin enlace Ethernet GDL_1–GDL_2 | 172.16.13.33 (Se0/1/0) | 2,682,112 |
| Además sin serial GDL_1–MEX | 172.16.23.125 (Se0/2/0) | 3,194,112 |

**Conclusión:** EIGRP elige la ruta con base en ancho de banda y retardo acumulado, no solo en el número de saltos. Los enlaces seriales tienen menos ancho de banda y más retardo que Gigabit Ethernet, por eso la métrica crece.

```
conf t
router eigrp 100
 network ...
```

---

## 1.8.1 — OSPF

**Objetivo:** configurar una red OSPF multiárea conectada a otro sistema autónomo (EIGRP), analizando tipos de rutas, vecindades y la base de datos de estado de enlace.

**Topología:** ZAPOPAN, GDL_1, GDL_2, MEX, MTY, QRO, R_INET y R_EIGRP. Proceso OSPF **99**.

| Área | Routers / redes |
|---|---|
| Área 0 (backbone) | MTY, GDL_1 (WAN), MEX (WAN) |
| Área 1 | ZAPOPAN, GDL_1 y GDL_2 (LAN 30.30.30.0/24 y 50.50.50.0/24) |
| Área 2 | MEX (LAN), GDL_2 (serial), QRO |
| Fuera de OSPF | R_INET (default route), R_EIGRP (EIGRP 100) |

**Router IDs**

| Router | Router ID |
|---|---|
| ZAPOPAN | 50.50.50.50 |
| GDL_1 | 172.16.23.126 |
| GDL_2 | 192.168.64.65 |
| QRO | 192.168.64.66 |
| MEX | 192.168.14.13 |
| MTY | 204.124.40.10 |

**Etapas y hallazgos**

1. **Área 1:** en el segmento 30.30.30.0/24, ZAPOPAN es DR, GDL_1 es BDR y GDL_2 es DROther.
2. **Áreas 0 y 2:** se identificaron los ABR con `show ip ospf border-routers` y se comprobó conectividad entre áreas con `ping` desde 50.50.50.55 hacia MTY, MEX y QRO. Antes de agregar la LAN de MEX a OSPF, el ping a 10.10.10.11 fallaba.
3. **LSAs:** cada área contiene LSA tipo 1 y 2; los routers de backbone muestran además LSA tipo 3 (resumen entre áreas).
4. **Salida a Internet:** default route en MTY hacia R_INET, propagada con `default-information originate`. Aparece como `O*E2 0.0.0.0/0` en los routers OSPF y como LSA tipo 5 en la base de datos.
5. **Integración con EIGRP:** redistribución en ambos sentidos en QRO. Las rutas hacia 80.80.80.0/24 aparecen en OSPF como **E1**, con costo 144 en GDL_2.

```
! Salida a Internet (MTY)
ip route 0.0.0.0 0.0.0.0 204.124.40.9
router ospf 99
 default-information originate

! Redistribución (QRO)
router ospf 99
 redistribute eigrp 100 metric 80 metric-type 1 subnets
router eigrp 100
 redistribute ospf 99 metric 2000 1000 255 1 1500
```

---

## Comparativa de protocolos

| Característica | Estático | RIP | EIGRP | OSPF |
|---|---|---|---|---|
| Distancia administrativa | 1 (default) | 120 | 90 | 110 |
| Métrica | — | Saltos | Ancho de banda + retardo | Costo |
| Soporta VLSM | Sí | Solo RIPv2 | Sí | Sí |
| Adaptación a fallas | Manual (flotantes como respaldo) | Automática | Automática | Automática |
| Diseño jerárquico | No | No | No | Sí (áreas) |

## Cómo reproducir

1. Abrir el archivo `.pkt` de la práctica (o armar la topología según el PDF) en Cisco Packet Tracer.
2. Cargar las configuraciones de cada router desde la carpeta `configs/` o copiarlas de los PDF.
3. Verificar con `show ip route`, `show ip protocols`, `show ip ospf neighbor`, `ping` y `tracert`.
4. Para simular fallas, usar `shutdown` / `no shutdown` sobre las interfaces indicadas en cada práctica.

## Aprendizajes del equipo

- Ver cómo cada configuración se refleja de inmediato en la tabla de enrutamiento hace más claros los conceptos teóricos.
- Configurar las direcciones IP no basta: las redes deben anunciarse correctamente en cada protocolo.
- Cada protocolo tiene su lugar: las rutas estáticas son simples, RIP es fácil pero limitado, EIGRP considera más factores en su métrica y OSPF escala mejor con áreas.
- Combinar OSPF con EIGRP mediante redistribución permite integrar redes de distintos proveedores.

## Créditos

Tecnológico de Monterrey — Integración de seguridad informática en redes y sistemas de software (Gpo 421). Profesora: Edith Carolina Arias Serna.