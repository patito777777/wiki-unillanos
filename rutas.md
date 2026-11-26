---
layout: default
title: "Horarios reales de rutas Unillanos 2026-2 (Barcelona y San Antonio)"
description: "Horarios reales y actualizados de las rutas (buses institucionales) de la Universidad de los Llanos: última ruta, rutas nocturnas, rutas de barrio y paraderos, sede Barcelona y San Antonio, semestre 2026-2. Incluye las diferencias con la información oficial."
permalink: /rutas/
---
{% assign r = site.data.rutas %}
{% assign parque = r.ruta_parque %}
{% assign bcn = r.rutas_nocturnas.sede_barcelona %}
{% assign sa = r.rutas_nocturnas.sede_san_antonio %}
{% assign barrios_data = r.rutas_barrio %}

# Rutas institucionales Unillanos ({{ r.meta.vigencia }})

**Vigencia:** {{ r.meta.vigencia }} · **Última actualización:** {{ r.meta.ultima_actualizacion }} · **Empresa:** {{ r.meta.empresa_transporte }}

> {{ r.meta.fuente }}

Horarios de los buses (rutas institucionales, transporte estudiantil) de la {{ r.meta.universidad }}, sede Barcelona y sede San Antonio, para el semestre {{ r.meta.vigencia }}.

## Última ruta del día

En la sede Barcelona, la última ruta regular sale a las {{ parque.entre_semana.salida_universidad.ultima }} ({{ parque.entre_semana.dias }}), y los sábados a las {{ parque.sabado.salida_universidad.ultima }}. El último bus de barrio sale a las {{ barrios_data.horarios_base.entre_semana.universidad_a_barrio | last }} entre semana. Si hay clase hasta tarde, existen rutas nocturnas a las {{ bcn.salidas[0].hora }} ({{ bcn.salidas[0].dias | join: " a " }}) y a las {{ bcn.salidas[1].hora }} (solo {{ bcn.salidas[1].dias | join: ", " }}); ver "Rutas nocturnas". Desde la {{ sa.salen_de }}, los buses a los barrios salen a las {{ sa.hora_barrios }} ({{ sa.dias }}).

## Avisos

{% for aviso in r.avisos -%}
- {{ aviso }}
{% endfor %}

## Diferencias con la información oficial

{% for dif in r.diferencias_oficial -%}
- **{{ dif.titulo }}:** {{ dif.detalle }}
{% endfor %}

## Calendario del semestre

{% for cal in r.calendario_semestre -%}
- **Semanas {{ cal.semanas }}:** {{ cal.rutas }}.
{% endfor %}

## {{ parque.nombre }}

{{ parque.descripcion }}

- **Entre semana, desde Villavicencio:** de {{ parque.entre_semana.salida_villavicencio.primera }} a {{ parque.entre_semana.salida_villavicencio.ultima }}, cada {{ parque.entre_semana.salida_villavicencio.cada_min }} min.
- **Entre semana, desde la universidad ({{ parque.punto_salida_universidad }}):** de {{ parque.entre_semana.salida_universidad.primera }} a {{ parque.entre_semana.salida_universidad.ultima }}, cada {{ parque.entre_semana.salida_universidad.cada_min }} min. {{ parque.entre_semana.salida_universidad.nota }}
- **Sábado, desde Villavicencio:** de {{ parque.sabado.salida_villavicencio[0].desde }} a {{ parque.sabado.salida_villavicencio[0].hasta }} cada {{ parque.sabado.salida_villavicencio[0].cada_min }} min, y de {{ parque.sabado.salida_villavicencio[1].desde }} a {{ parque.sabado.salida_villavicencio[1].hasta }} cada {{ parque.sabado.salida_villavicencio[1].cada_min }} min.
- **Sábado, desde la universidad:** de {{ parque.sabado.salida_universidad.primera }} a {{ parque.sabado.salida_universidad.ultima }}, cada {{ parque.sabado.salida_universidad.cada_min }} min. {{ parque.sabado.salida_universidad.nota }}

**Paraderos (barrio a universidad):**

{% for p in parque.paraderos_barrio_a_universidad -%}
{{ forloop.index }}. {{ p.nombre }}{% if p.nota %}, {{ p.nota }}{% endif %}
{% endfor %}

{{ parque.sentido_universidad_a_barrio }}

## Rutas nocturnas

### Sede Barcelona

{{ bcn.descripcion }}

- **Punto de salida:** {{ bcn.punto_salida }}.
- **Condición:** {{ bcn.condicion }}.
{% for s in bcn.salidas -%}
- **{{ s.hora }}:** {{ s.dias | join: ", " }}.{% if s.nota %} {{ s.nota }}{% endif %}
{% endfor %}

{{ bcn.nota }}

### Sede San Antonio

{{ sa.descripcion }}

- **Hacia el parque de los estudiantes:** de {{ sa.hacia_el_parque.desde }} a {{ sa.hacia_el_parque.hasta }}, cada {{ sa.hacia_el_parque.cada_min }} min ({{ sa.hacia_el_parque.total_rutas }} rutas en total: {{ sa.hacia_el_parque.salidas | join: ", " }}), {{ sa.hacia_el_parque.nota }}.
- **Hacia los barrios:** salen de la {{ sa.salen_de }} a las {{ sa.hora_barrios }}, de {{ sa.dias }}.

{% for rb in sa.rutas_a_barrios -%}
**{{ rb.nombre }} ({{ rb.hora }}):**

{% for par in rb.paraderos -%}
{{ forloop.index }}. {{ par }}
{% endfor %}

{% endfor -%}
## Rutas de barrio

{{ barrios_data.descripcion }}

{% for n in barrios_data.notas_generales -%}
- {{ n }}
{% endfor %}

### Horarios base (entre semana)

- **Barrio a universidad:** {{ barrios_data.horarios_base.entre_semana.barrio_a_universidad | join: ", " }}
- **Universidad a barrio:** {{ barrios_data.horarios_base.entre_semana.universidad_a_barrio | join: ", " }}

### Resumen por barrio

| Barrio | Ida (entre semana) | Vuelta (entre semana) | Sábado |
|---|---|---|---|
{% for b in barrios_data.barrios -%}
| [{{ b.nombre }}](#{{ b.id }}) | {{ b.horarios.entre_semana.barrio_a_universidad | join: ", " }} | {{ b.horarios.entre_semana.universidad_a_barrio | join: ", " }} | {% if b.horarios.sabado %}Ida {{ b.horarios.sabado.barrio_a_universidad | join: ", " }}; vuelta {{ b.horarios.sabado.universidad_a_barrio | join: ", " }}{% else %}No hay ruta{% endif %} |
{% endfor %}

{% for b in barrios_data.barrios -%}
### {{ b.nombre }} {#{{ b.id }}}

**Paraderos (barrio a universidad):**

{% for par in b.paraderos -%}
{{ forloop.index }}. {{ par }}
{% endfor %}

{% if b.paradero_extra_universidad_a_barrio -%}
**Paradero extra:** {{ b.paradero_extra_universidad_a_barrio }}

{% endif -%}
{% if b.notas.size > 0 -%}
**Aclaraciones:**

{% for nota in b.notas -%}
- {{ nota }}
{% endfor %}

{% endif -%}
{% endfor -%}
