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

  const formatSingleTime = (hhmm) => {
    const match = typeof hhmm === "string" && hhmm.match(/^(\d{1,2}):(\d{2})$/);
    if (!match) return hhmm || "";
    const h24 = parseInt(match[1], 10);
    const mins = match[2];
    const suffix = h24 >= 12 ? "p. m." : "a. m.";
    const h12 = h24 % 12 === 0 ? 12 : h24 % 12;
    return `${h12}:${mins} ${suffix}`;
  };

  // Convierte horas sueltas HH:MM o rangos HH:MM a HH:MM dentro de cualquier texto
  const formatTextTimes = (text) => {
    if (typeof text !== "string" || !text) return "";
    return text.replace(/\b(\d{1,2}:\d{2})\b/g, (m) => formatSingleTime(m));
  };

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

    const joinList = (arr, sep = ", ") =>
      Array.isArray(arr) && arr.length ? arr.map(formatTextTimes).join(sep) : "—";
    const olItems = (arr, formatter) =>
      "<ol>" + (arr || []).map((item, i) => `<li>${formatter(item, i)}</li>`).join("") + "</ol>";
    const ulItems = (arr, formatter) =>
      "<ul>" + (arr || []).map((item, i) => `<li>${formatter(item, i)}</li>`).join("") + "</ul>";

    const sabVil = sab.salida_villavicencio || [];
    const sabUni = sab.salida_universidad || {};
    const saParque = sa.hacia_el_parque || {};
    const baseEs = (barriosData.horarios_base && barriosData.horarios_base.entre_semana) || {};
    const ultimaBarrio = Array.isArray(baseEs.universidad_a_barrio)
      ? formatSingleTime(baseEs.universidad_a_barrio[baseEs.universidad_a_barrio.length - 1])
      : "—";

    const bcnDias0 = (bcn.salidas && bcn.salidas[0] && bcn.salidas[0].dias) || [];
    const bcnRango0 = bcnDias0.length
      ? `${bcnDias0[0]} a ${bcnDias0[bcnDias0.length - 1]}`
      : "";

    const filasBarrios = barrios
      .map((b) => {
        const hEs = (b.horarios && b.horarios.entre_semana) || {};
        const hSab = b.horarios && b.horarios.sabado;
        const sabCell = hSab
          ? `Ida ${joinList(hSab.barrio_a_universidad)}; vuelta ${joinList(hSab.universidad_a_barrio)}`
          : "No hay ruta";
        return `
          <tr>
            <td><a href="#${b.id}">${formatTextTimes(b.nombre)}</a></td>
            <td>${joinList(hEs.barrio_a_universidad)}</td>
            <td>${joinList(hEs.universidad_a_barrio)}</td>
            <td>${sabCell}</td>
          </tr>`;
      })
      .join("");

    const detalleBarrios = barrios
      .map((b) => {
        const extra = b.paradero_extra_universidad_a_barrio
          ? `<p><strong>Paradero extra:</strong> ${formatTextTimes(b.paradero_extra_universidad_a_barrio)}</p>`
          : "";
        const notas =
          Array.isArray(b.notas) && b.notas.length
            ? `<p><strong>Aclaraciones:</strong></p>${ulItems(b.notas, (n) => formatTextTimes(n))}`
            : "";
        return `
          <h3 id="${b.id}">${formatTextTimes(b.nombre)}</h3>
          <p><strong>Paraderos (barrio a universidad):</strong></p>
          ${olItems(b.paraderos, (par) => formatTextTimes(par))}
          ${extra}
          ${notas}`;
      })
      .join("");

    const saBarriosHtml = (sa.rutas_a_barrios || [])
      .map(
        (rb) => `
          <p><strong>${formatTextTimes(rb.nombre)} (${formatSingleTime(rb.hora)}):</strong></p>
          ${olItems(rb.paraderos, (par) => formatTextTimes(par))}`
      )
      .join("");

    container.innerHTML = `
      <h1>Rutas institucionales Unillanos (${meta.vigencia || ""})</h1>
      <p><strong>Vigencia:</strong> ${meta.vigencia || ""} · <strong>Última actualización:</strong> ${meta.ultima_actualizacion || ""} · <strong>Empresa:</strong> ${meta.empresa_transporte || ""}</p>
      <blockquote><p>${formatTextTimes(meta.fuente || "")}</p></blockquote>
      <p>Horarios de los buses (rutas institucionales, transporte estudiantil) de la ${meta.universidad || ""}, sede Barcelona y sede San Antonio, para el semestre ${meta.vigencia || ""}.</p>

      <h2>Última ruta del día</h2>
      <p>En la sede Barcelona, la última ruta regular sale a las ${formatSingleTime((es.salida_universidad && es.salida_universidad.ultima) || "")} (${es.dias || ""}), y los sábados a las ${formatSingleTime(sabUni.ultima || "")}. El último bus de barrio sale a las ${ultimaBarrio} entre semana. Si hay clase hasta tarde, existen rutas nocturnas a las ${formatSingleTime((bcn.salidas && bcn.salidas[0] && bcn.salidas[0].hora) || "")} (${bcnRango0}) y a las ${formatSingleTime((bcn.salidas && bcn.salidas[1] && bcn.salidas[1].hora) || "")} (solo ${joinList(bcn.salidas && bcn.salidas[1] && bcn.salidas[1].dias)}); ver "Rutas nocturnas". Desde la ${sa.salen_de || ""}, los buses a los barrios salen a las ${formatSingleTime(sa.hora_barrios || "")} (${sa.dias || ""}).</p>

      <h2>Avisos</h2>
      ${ulItems(r.avisos, (a) => formatTextTimes(a))}

      <h2>Diferencias con la información oficial</h2>
      ${ulItems(r.diferencias_oficial, (d) => `<strong>${formatTextTimes(d.titulo)}:</strong> ${formatTextTimes(d.detalle)}`)}

      <h2>Calendario del semestre</h2>
      ${ulItems(r.calendario_semestre, (c) => `<strong>Semanas ${formatTextTimes(c.semanas)}:</strong> ${formatTextTimes(c.rutas)}.`)}

      <h2>${formatTextTimes(parque.nombre || "")}</h2>
      <p>${formatTextTimes(parque.descripcion || "")}</p>
      <ul>
        <li><strong>Entre semana, desde Villavicencio:</strong> de ${formatSingleTime((es.salida_villavicencio && es.salida_villavicencio.primera) || "")} a ${formatSingleTime((es.salida_villavicencio && es.salida_villavicencio.ultima) || "")}, cada ${(es.salida_villavicencio && es.salida_villavicencio.cada_min) || ""} min.</li>
        <li><strong>Entre semana, desde la universidad (${formatTextTimes(parque.punto_salida_universidad || "")}):</strong> de ${formatSingleTime((es.salida_universidad && es.salida_universidad.primera) || "")} a ${formatSingleTime((es.salida_universidad && es.salida_universidad.ultima) || "")}, cada ${(es.salida_universidad && es.salida_universidad.cada_min) || ""} min. ${formatTextTimes((es.salida_universidad && es.salida_universidad.nota) || "")}</li>
        <li><strong>Sábado, desde Villavicencio:</strong> de ${formatSingleTime((sabVil[0] && sabVil[0].desde) || "")} a ${formatSingleTime((sabVil[0] && sabVil[0].hasta) || "")} cada ${(sabVil[0] && sabVil[0].cada_min) || ""} min, y de ${formatSingleTime((sabVil[1] && sabVil[1].desde) || "")} a ${formatSingleTime((sabVil[1] && sabVil[1].hasta) || "")} cada ${(sabVil[1] && sabVil[1].cada_min) || ""} min.</li>
        <li><strong>Sábado, desde la universidad:</strong> de ${formatSingleTime(sabUni.primera || "")} a ${formatSingleTime(sabUni.ultima || "")}, cada ${sabUni.cada_min || ""} min. ${formatTextTimes(sabUni.nota || "")}</li>
      </ul>

      <p><strong>Paraderos (barrio a universidad):</strong></p>
      ${olItems(parque.paraderos_barrio_a_universidad, (p) => `${formatTextTimes(p.nombre)}${p.nota ? ", " + formatTextTimes(p.nota) : ""}`)}
      <p>${formatTextTimes(parque.sentido_universidad_a_barrio || "")}</p>

      <h2>Rutas nocturnas</h2>
      <h3>Sede Barcelona</h3>
      <p>${formatTextTimes(bcn.descripcion || "")}</p>
      <ul>
        <li><strong>Punto de salida:</strong> ${formatTextTimes(bcn.punto_salida || "")}.</li>
        <li><strong>Condición:</strong> ${formatTextTimes(bcn.condicion || "")}.</li>
        ${(bcn.salidas || [])
          .map((s) => `<li><strong>${formatSingleTime(s.hora)}:</strong> ${joinList(s.dias)}.${s.nota ? " " + formatTextTimes(s.nota) : ""}</li>`)
          .join("")}
      </ul>
      <p>${formatTextTimes(bcn.nota || "")}</p>

      <h3>Sede San Antonio</h3>
      <p>${formatTextTimes(sa.descripcion || "")}</p>
      <ul>
        <li><strong>Hacia el parque de los estudiantes:</strong> de ${formatSingleTime(saParque.desde || "")} a ${formatSingleTime(saParque.hasta || "")}, cada ${saParque.cada_min || ""} min (${saParque.total_rutas || ""} rutas en total: ${joinList(saParque.salidas)}), ${formatTextTimes(saParque.nota || "")}.</li>
        <li><strong>Hacia los barrios:</strong> salen de la ${sa.salen_de || ""} a las ${formatSingleTime(sa.hora_barrios || "")}, de ${sa.dias || ""}.</li>
      </ul>
      ${saBarriosHtml}

      <h2>Rutas de barrio</h2>
      <p>${formatTextTimes(barriosData.descripcion || "")}</p>
      ${ulItems(barriosData.notas_generales, (n) => formatTextTimes(n))}

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
