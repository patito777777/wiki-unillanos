---
layout: default
title: "Horarios reales de rutas Unillanos 2026-2 (Barcelona y San Antonio)"
description: "Horarios reales y actualizados de las rutas (buses institucionales) de la Universidad de los Llanos: última ruta, rutas nocturnas, rutas de barrio y paraderos, sede Barcelona y San Antonio, semestre 2026-2. Incluye las diferencias con la información oficial."
permalink: /rutas/
---

<div id="rutas-app">Cargando horarios desde rutas.json...</div>

<script>
(async function () {
  const container = document.getElementById("rutas-app");
  const jsonUrl = "{{ '/rutas.json' | relative_url }}";

  try {
    const res = await fetch(jsonUrl, { cache: "no-store" });
    if (!res.ok) throw new Error("HTTP " + res.status);
    const r = await res.json();

    const meta = r.meta || {};
    const parque = r.ruta_parque || {};
    const es = parque.entre_semana || {};
    const sab = parque.sabado || {};
    const bcn = (r.rutas_nocturnas && r.rutas_nocturnas.sede_barcelona) || {};
    const sa = (r.rutas_nocturnas && r.rutas_nocturnas.sede_san_antonio) || {};
    const barriosData = r.rutas_barrio || {};
    const barrios = barriosData.barrios || [];

    const joinList = (arr, sep = ", ") => (Array.isArray(arr) && arr.length ? arr.join(sep) : "—");
    const olItems = (arr, formatter) =>
      "<ol>" + (arr || []).map((item, i) => `<li>${formatter(item, i)}</li>`).join("") + "</ol>";
    const ulItems = (arr, formatter) =>
      "<ul>" + (arr || []).map((item, i) => `<li>${formatter(item, i)}</li>`).join("") + "</ul>";

    const sabVil = sab.salida_villavicencio || [];
    const sabUni = sab.salida_universidad || {};
    const saParque = sa.hacia_el_parque || {};
    const baseEs = (barriosData.horarios_base && barriosData.horarios_base.entre_semana) || {};
    const ultimaBarrio = Array.isArray(baseEs.universidad_a_barrio)
      ? baseEs.universidad_a_barrio[baseEs.universidad_a_barrio.length - 1]
      : "—";

    const filasBarrios = barrios
      .map((b) => {
        const hEs = (b.horarios && b.horarios.entre_semana) || {};
        const hSab = b.horarios && b.horarios.sabado;
        const sabCell = hSab
          ? `Ida ${joinList(hSab.barrio_a_universidad)}; vuelta ${joinList(hSab.universidad_a_barrio)}`
          : "No hay ruta";
        return `
          <tr>
            <td><a href="#${b.id}">${b.nombre}</a></td>
            <td>${joinList(hEs.barrio_a_universidad)}</td>
            <td>${joinList(hEs.universidad_a_barrio)}</td>
            <td>${sabCell}</td>
          </tr>`;
      })
      .join("");

    const detalleBarrios = barrios
      .map((b) => {
        const extra = b.paradero_extra_universidad_a_barrio
          ? `<p><strong>Paradero extra:</strong> ${b.paradero_extra_universidad_a_barrio}</p>`
          : "";
        const notas =
          Array.isArray(b.notas) && b.notas.length
            ? `<p><strong>Aclaraciones:</strong></p>${ulItems(b.notas, (n) => n)}`
            : "";
        return `
          <h3 id="${b.id}">${b.nombre}</h3>
          <p><strong>Paraderos (barrio a universidad):</strong></p>
          ${olItems(b.paraderos, (par) => par)}
          ${extra}
          ${notas}`;
      })
      .join("");

    const saBarriosHtml = (sa.rutas_a_barrios || [])
      .map(
        (rb) => `
          <p><strong>${rb.nombre} (${rb.hora}):</strong></p>
          ${olItems(rb.paraderos, (par) => par)}`
      )
      .join("");

    container.innerHTML = `
      <h1>Rutas institucionales Unillanos (${meta.vigencia || ""})</h1>
      <p><strong>Vigencia:</strong> ${meta.vigencia || ""} · <strong>Última actualización:</strong> ${meta.ultima_actualizacion || ""} · <strong>Empresa:</strong> ${meta.empresa_transporte || ""}</p>
      <blockquote><p>${meta.fuente || ""}</p></blockquote>
      <p>Horarios de los buses (rutas institucionales, transporte estudiantil) de la ${meta.universidad || ""}, sede Barcelona y sede San Antonio, para el semestre ${meta.vigencia || ""}.</p>

      <h2>Última ruta del día</h2>
      <p>En la sede Barcelona, la última ruta regular sale a las ${(es.salida_universidad && es.salida_universidad.ultima) || ""} (${es.dias || ""}), y los sábados a las ${sabUni.ultima || ""}. El último bus de barrio sale a las ${ultimaBarrio} entre semana. Si hay clase hasta tarde, existen rutas nocturnas a las {{ bcn.salidas[0].hora }} ({{ bcn.salidas[0].dias | first }} a {{ bcn.salidas[0].dias | last }}) y a las {{ bcn.salidas[1].hora }} (solo {{ bcn.salidas[1].dias | join: ", " }}); ver "Rutas nocturnas". Desde la ${sa.salen_de || ""}, los buses a los barrios salen a las ${sa.hora_barrios || ""} (${sa.dias || ""}).</p>

      <h2>Avisos</h2>
      ${ulItems(r.avisos, (a) => a)}

      <h2>Diferencias con la información oficial</h2>
      ${ulItems(r.diferencias_oficial, (d) => `<strong>${d.titulo}:</strong> ${d.detalle}`)}

      <h2>Calendario del semestre</h2>
      ${ulItems(r.calendario_semestre, (c) => `<strong>Semanas ${c.semanas}:</strong> ${c.rutas}.`)}

      <h2>${parque.nombre || ""}</h2>
      <p>${parque.descripcion || ""}</p>
      <ul>
        <li><strong>Entre semana, desde Villavicencio:</strong> de ${(es.salida_villavicencio && es.salida_villavicencio.primera) || ""} a ${(es.salida_villavicencio && es.salida_villavicencio.ultima) || ""}, cada ${(es.salida_villavicencio && es.salida_villavicencio.cada_min) || ""} min.</li>
        <li><strong>Entre semana, desde la universidad (${parque.punto_salida_universidad || ""}):</strong> de ${(es.salida_universidad && es.salida_universidad.primera) || ""} a ${(es.salida_universidad && es.salida_universidad.ultima) || ""}, cada ${(es.salida_universidad && es.salida_universidad.cada_min) || ""} min. ${(es.salida_universidad && es.salida_universidad.nota) || ""}</li>
        <li><strong>Sábado, desde Villavicencio:</strong> de ${(sabVil[0] && sabVil[0].desde) || ""} a ${(sabVil[0] && sabVil[0].hasta) || ""} cada ${(sabVil[0] && sabVil[0].cada_min) || ""} min, y de ${(sabVil[1] && sabVil[1].desde) || ""} a ${(sabVil[1] && sabVil[1].hasta) || ""} cada ${(sabVil[1] && sabVil[1].cada_min) || ""} min.</li>
        <li><strong>Sábado, desde la universidad:</strong> de ${sabUni.primera || ""} a ${sabUni.ultima || ""}, cada ${sabUni.cada_min || ""} min. ${sabUni.nota || ""}</li>
      </ul>

      <p><strong>Paraderos (barrio a universidad):</strong></p>
      ${olItems(parque.paraderos_barrio_a_universidad, (p) => `${p.nombre}${p.nota ? ", " + p.nota : ""}`)}
      <p>${parque.sentido_universidad_a_barrio || ""}</p>

      <h2>Rutas nocturnas</h2>
      <h3>Sede Barcelona</h3>
      <p>${bcn.descripcion || ""}</p>
      <ul>
        <li><strong>Punto de salida:</strong> ${bcn.punto_salida || ""}.</li>
        <li><strong>Condición:</strong> ${bcn.condicion || ""}.</li>
        ${(bcn.salidas || [])
          .map((s) => `<li><strong>${s.hora}:</strong> ${joinList(s.dias)}.${s.nota ? " " + s.nota : ""}</li>`)
          .join("")}
      </ul>
      <p>${bcn.nota || ""}</p>

      <h3>Sede San Antonio</h3>
      <p>${sa.descripcion || ""}</p>
      <ul>
        <li><strong>Hacia el parque de los estudiantes:</strong> de ${saParque.desde || ""} a ${saParque.hasta || ""}, cada ${saParque.cada_min || ""} min (${saParque.total_rutas || ""} rutas en total: ${joinList(saParque.salidas)}), ${saParque.nota || ""}.</li>
        <li><strong>Hacia los barrios:</strong> salen de la ${sa.salen_de || ""} a las ${sa.hora_barrios || ""}, de ${sa.dias || ""}.</li>
      </ul>
      ${saBarriosHtml}

      <h2>Rutas de barrio</h2>
      <p>${barriosData.descripcion || ""}</p>
      ${ulItems(barriosData.notas_generales, (n) => n)}

      <h3>Horarios base (entre semana)</h3>
      <ul>
        <li><strong>Barrio a universidad:</strong> ${joinList(baseEs.barrio_a_universidad)}</li>
        <li><strong>Universidad a barrio:</strong> ${joinList(baseEs.universidad_a_barrio)}</li>
      </ul>

      <h3>Resumen por barrio</h3>
      <table>
        <thead>
          <tr>
            <th>Barrio</th>
            <th>Ida (entre semana)</th>
            <th>Vuelta (entre semana)</th>
            <th>Sábado</th>
          </tr>
        </thead>
        <tbody>${filasBarrios}</tbody>
      </table>

      ${detalleBarrios}
    `;
  } catch (err) {
    container.innerHTML = `<p><strong>Error al cargar rutas.json:</strong> ${err.message}</p>`;
  }
})();
</script>
