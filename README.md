<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Presupuesto mensual mínimo</title>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:wght@600;700&family=Nunito+Sans:wght@400;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.3.1/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.3.1/umd/react-dom.production.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@babel/standalone@7.24.7/babel.min.js"></script>
<style>
:root{--bg:#F6EEE0;--card:#FFFAF2;--ink:#3E2C20;--mute:#8A7566;--line:#E6D8C3;--track:#EBDDC8;--red:#B5483A;
 box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#2A201A;--card:#352A22;--ink:#F3E8D8;--mute:#BBA793;--line:#4A3B30;--track:#463829;--red:#E0735F}}
:root[data-theme="dark"]{--bg:#2A201A;--card:#352A22;--ink:#F3E8D8;--mute:#BBA793;--line:#4A3B30;--track:#463829;--red:#E0735F}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
body{background:var(--bg);color:var(--ink);font-family:'Nunito Sans',system-ui,sans-serif;margin:0}
h1,h2,.num{font-family:'Fraunces',Georgia,serif}
.card{background:var(--card);border:1px solid var(--line);border-radius:18px}
input{font-family:inherit;color:var(--ink)}
input:focus-visible,button:focus-visible{outline:2px solid #C98B6B;outline-offset:2px}
</style>
</head>
<body>
<div id="root"></div>
<script type="text/babel">
const { useState, useMemo } = React;

/* ---------- Utilidades ---------- */
const cop = n => "$" + Math.round(n).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
const fmt = n => n ? Math.round(n).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".") : "";
const parse = s => parseInt(String(s).replace(/\D/g, ""), 10) || 0;

/* Iconos (trazos estilo Lucide) */
const P = {
  wallet: ["M19 7V4a1 1 0 0 0-1-1H5a2 2 0 0 0 0 4h15a1 1 0 0 1 1 1v4h-3a2 2 0 0 0 0 4h3a1 1 0 0 0 1-1v-2a1 1 0 0 0-1-1", "M3 5v14a2 2 0 0 0 2 2h15a1 1 0 0 0 1-1v-4"],
  home: ["M3 10l9-7 9 7v10a1 1 0 0 1-1 1h-5v-7H9v7H4a1 1 0 0 1-1-1z"],
  save: ["M12 2v20", "M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"],
  heart: ["M19 14c1.49-1.46 3-3.21 3-5.5A5.5 5.5 0 0 0 16.5 3c-1.76 0-3 .5-4.5 2-1.5-1.5-2.74-2-4.5-2A5.5 5.5 0 0 0 2 8.5c0 2.3 1.5 4.05 3 5.5l7 7Z"],
  users: ["M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2", "M9 11a4 4 0 1 0 0-8 4 4 0 0 0 0 8", "M22 21v-2a4 4 0 0 0-3-3.87", "M16 3.13a4 4 0 0 1 0 7.75"],
  smile: ["M12 2a10 10 0 1 0 0 20 10 10 0 0 0 0-20z", "M8 14s1.5 2 4 2 4-2 4-2", "M9 9h.01", "M15 9h.01"],
  star: ["M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01z"],
  reset: ["M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8", "M3 3v5h5"],
  chev: ["M6 9l6 6 6-6"],
  car: ["M19 17h2c.6 0 1-.4 1-1v-3c0-.9-.7-1.7-1.5-1.9C18.7 10.6 16 10 16 10s-1.3-1.4-2.2-2.3c-.5-.4-1.1-.7-1.8-.7H5c-.6 0-1.1.4-1.4.9l-1.4 2.9A3.7 3.7 0 0 0 2 12v4c0 .6.4 1 1 1h2", "M9 17m-2 0a2 2 0 1 0 4 0a2 2 0 1 0-4 0", "M17 17m-2 0a2 2 0 1 0 4 0a2 2 0 1 0-4 0", "M9 17h6"],
  alert: ["M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z", "M12 9v4", "M12 17h.01"],
};
const Icon = ({ n, s = 20 }) => (
  <svg width={s} height={s} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" aria-hidden="true">
    {P[n].map((d, i) => <path key={i} d={d} />)}
  </svg>
);

/* ---------- Plantillas ---------- */
const HOUSING = [
  { id: "arriendo", label: "Arriendo", share: 0.4 },
  { id: "servicios", label: "Servicios", share: 0.2 },
  { id: "comida", label: "Comida", share: 0.4 },
];
const CAR_ITEMS = [
  { id: "soat", label: "SOAT", def: 600000 },
  { id: "rtm", label: "Revisión técnico-mecánica", def: 300000 },
  { id: "otros", label: "Impuesto, mantenimiento e imprevistos", def: 0 },
];
const CAR_DEFAULT = Object.fromEntries(CAR_ITEMS.map(i => [i.id, i.def]));
const TEMPLATES = {
  t1: {
    name: "Real estrato 4 · 60/10/10/10/10",
    desc: "Parte del costo real de vida. El 2% del carro sale del bloque de gustos (10% → 8%).",
    cats: [
      { id: "vivienda", label: "Vivienda y básicos", pct: 60, icon: "home", color: "#8C6A4F", housing: true },
      { id: "ahorro", label: "Ahorro / libertad financiera", pct: 10, icon: "save", color: "#C9A27E" },
      { id: "carro", label: "Ahorro para el carro", pct: 2, icon: "car", color: "#A9703F", car: true },
      { id: "diezmo", label: "Diezmo / caridad", pct: 10, icon: "heart", color: "#E8A08A" },
      { id: "hijos", label: "Hijos / educación / cuidado", pct: 10, icon: "users", color: "#D4896F" },
      { id: "personal", label: "Personales / ocio", pct: 8, icon: "smile", color: "#B5483A" },
    ],
  },
  t2: {
    name: "Tradicional · T. Harv Eker adaptada",
    desc: "50% fijos y bolsillos de 10%. El 2% del carro sale de gustos (10% → 8%).",
    cats: [
      { id: "vivienda", label: "Gastos fijos", pct: 50, icon: "home", color: "#8C6A4F", housing: true },
      { id: "ahorro", label: "Ahorro", pct: 10, icon: "save", color: "#C9A27E" },
      { id: "carro", label: "Ahorro para el carro", pct: 2, icon: "car", color: "#A9703F", car: true },
      { id: "hijos", label: "Educación / hijos", pct: 10, icon: "users", color: "#D4896F" },
      { id: "diezmo", label: "Caridad / diezmo", pct: 10, icon: "heart", color: "#E8A08A" },
      { id: "proyectos", label: "Proyectos", pct: 10, icon: "star", color: "#A67B5B" },
      { id: "personal", label: "Gustos", pct: 8, icon: "smile", color: "#B5483A" },
    ],
  },
};

/* ---------- Campo de dinero ---------- */
function MoneyInput({ value, onChange, placeholder, label, big }) {
  return (
    <label className="flex items-center gap-1 rounded-xl px-3 py-2" style={{ background: "var(--bg)", border: "1px solid var(--line)" }}>
      <span className="sr-only">{label}</span>
      <span style={{ color: "var(--mute)" }}>$</span>
      <input
        inputMode="numeric" value={fmt(value)} placeholder={placeholder || "0"}
        onChange={e => onChange(parse(e.target.value))}
        className={"w-full bg-transparent text-right outline-none " + (big ? "text-xl font-bold" : "font-semibold")}
        style={{ minWidth: 0 }}
      />
    </label>
  );
}

/* ---------- Dona ---------- */
function Donut({ slices, base, active, setActive }) {
  const R = 70, C = 2 * Math.PI * R;
  let off = 0;
  const a = slices.find(s => s.id === active);
  return (
    <div className="relative mx-auto" style={{ width: 220, height: 220 }}>
      <svg viewBox="0 0 200 200" width="220" height="220" role="img" aria-label="Distribución del ingreso">
        <g transform="rotate(-90 100 100)">
          <circle cx="100" cy="100" r={R} fill="none" stroke="var(--track)" strokeWidth="28" />
          {slices.map(s => {
            const len = base ? Math.min(s.value / base, 1) * C : 0;
            const el = (
              <circle key={s.id} cx="100" cy="100" r={R} fill="none" stroke={s.color}
                strokeWidth={active === s.id ? 34 : 28}
                strokeDasharray={`${Math.max(len - 1.5, 0)} ${C}`} strokeDashoffset={-off}
                style={{ cursor: "pointer", transition: "stroke-width .15s" }}
                onMouseEnter={() => setActive(s.id)} onMouseLeave={() => setActive(null)}
                onClick={() => setActive(active === s.id ? null : s.id)} />
            );
            off += len;
            return el;
          })}
        </g>
      </svg>
      <div className="absolute inset-0 flex flex-col items-center justify-center text-center px-10 pointer-events-none">
        <span className="text-xs" style={{ color: "var(--mute)" }}>{a ? a.label : "Asignado"}</span>
        <span className="num text-xl font-bold">{a ? cop(a.value) : base ? Math.round(slices.reduce((t, s) => t + s.value, 0) / base * 100) + "%" : "—"}</span>
        {a && <span className="text-xs" style={{ color: "var(--mute)" }}>{base ? Math.round(a.value / base * 100) : 0}% del ingreso</span>}
      </div>
    </div>
  );
}

/* ---------- App ---------- */
function App() {
  const [income, setIncome] = useState(5000000);
  const [variable, setVariable] = useState(0);
  const [tpl, setTpl] = useState("t1");
  const [over, setOver] = useState({});      // ajustes manuales por plantilla: {t1:{ahorro:n, "vivienda.arriendo":n}}
  const [open, setOpen] = useState(true);
  const [active, setActive] = useState(null);
  const [cash, setCash] = useState(0);        // efectivo disponible
  const [bank, setBank] = useState(0);        // saldo en cuenta de ahorro
  const [car, setCar] = useState(CAR_DEFAULT);   // costo anual de cada gasto del carro

  const base = income + variable;
  const T = TEMPLATES[tpl];
  const ov = over[tpl] || {};
  const setVal = (k, v) => setOver(o => ({ ...o, [tpl]: { ...(o[tpl] || {}), [k]: v } }));
  const reset = () => { setIncome(5000000); setVariable(0); setOver({}); setCar(CAR_DEFAULT); setCash(0); setBank(0); };

  const rows = useMemo(() => T.cats.map(c => {
    const target = base * c.pct / 100;
    if (c.car) {
      const yr = Object.values(car).reduce((a, b) => a + b, 0);
      return { ...c, target, value: Math.round(yr / 12) };
    }
    if (c.housing) {
      const subs = HOUSING.map(h => ({ ...h, value: ov["vivienda." + h.id] ?? Math.round(target * h.share) }));
      return { ...c, target, subs, value: subs.reduce((t, s) => t + s.value, 0) };
    }
    return { ...c, target, value: ov[c.id] ?? Math.round(target) };
  }), [tpl, base, over, car]);

  const assigned = rows.reduce((t, r) => t + r.value, 0);
  const left = base - assigned;
  const pctOf = v => base ? v / base * 100 : 0;

  const stat = (label, value, tone) => (
    <div className="card p-4 flex-1 min-w-[150px]">
      <div className="text-sm" style={{ color: "var(--mute)" }}>{label}</div>
      <div className="num text-2xl font-bold mt-1" style={{ color: tone }}>{value}</div>
    </div>
  );

  return (
    <main className="max-w-5xl mx-auto px-4 py-6 sm:py-10">
      <header className="flex flex-wrap items-end justify-between gap-3 mb-6">
        <div>
          <h1 className="text-3xl sm:text-4xl font-bold">Presupuesto del mes</h1>
          <p style={{ color: "var(--mute)" }}>Ponga su ingreso y reparta. Todo se recalcula al instante.</p>
        </div>
        <button onClick={reset} className="flex items-center gap-2 rounded-full px-4 py-2 font-semibold" style={{ background: "#A67B5B", color: "#FFF8EE" }}>
          <Icon n="reset" s={16} /> Restablecer valores
        </button>
      </header>

      {/* Ingresos */}
      <section className="card p-4 sm:p-5 mb-4 grid gap-4 sm:grid-cols-2">
        <div>
          <div className="flex items-center gap-2 mb-2 font-semibold"><Icon n="wallet" s={18} /> Ingreso fijo mensual</div>
          <MoneyInput big label="Ingreso fijo mensual" value={income} onChange={setIncome} />
        </div>
        <div>
          <div className="flex items-center gap-2 mb-2 font-semibold"><Icon n="star" s={18} /> Ingresos variables (opcional)</div>
          <MoneyInput big label="Ingresos variables" value={variable} onChange={setVariable} placeholder="0" />
        </div>
      </section>

      {/* Plantillas */}
      <section className="mb-4" aria-label="Plantillas">
        <div className="grid gap-2 sm:grid-cols-2">
          {Object.entries(TEMPLATES).map(([k, t]) => (
            <button key={k} onClick={() => setTpl(k)} aria-pressed={tpl === k}
              className="text-left rounded-2xl p-4 transition"
              style={tpl === k ? { background: "#A67B5B", color: "#FFF8EE", border: "1px solid #A67B5B" } : { background: "var(--card)", border: "1px solid var(--line)" }}>
              <div className="font-bold">{t.name}</div>
              <div className="text-sm opacity-80">{t.desc}</div>
            </button>
          ))}
        </div>
      </section>

      {/* Resumen */}
      <section className="flex flex-wrap gap-3 mb-4">
        {stat("Total ingresos", cop(base), "var(--ink)")}
        {stat("Gastos asignados", cop(assigned), "#A67B5B")}
        {stat(left >= 0 ? "Saldo restante" : "Te pasaste por", cop(Math.abs(left)), left >= 0 ? "#8C6A4F" : "var(--red)")}
      </section>
      {left < 0 && (
        <div className="flex items-center gap-2 rounded-xl px-4 py-3 mb-4 font-semibold" style={{ background: "#F6D9D2", color: "#8E2F23" }} role="alert">
          <Icon n="alert" s={18} /> Lo asignado supera el ingreso. Reduzca alguna categoría.
        </div>
      )}

      <div className="grid gap-4 lg:grid-cols-5">
        {/* Categorías */}
        <section className="lg:col-span-3 grid gap-3 content-start">
          {rows.map(r => {
            const p = pctOf(r.value), over_ = r.value > r.target + 1;
            return (
              <article key={r.id} className="card p-4" onMouseEnter={() => setActive(r.id)} onMouseLeave={() => setActive(null)}
                style={active === r.id ? { borderColor: r.color } : null}>
                <div className="flex items-center gap-3">
                  <span className="grid place-items-center rounded-xl" style={{ width: 38, height: 38, background: r.color, color: "#FFF8EE" }}><Icon n={r.icon} /></span>
                  <div className="flex-1 min-w-0">
                    <div className="font-bold leading-tight">{r.label}</div>
                    <div className="text-sm" style={{ color: over_ ? "var(--red)" : "var(--mute)" }}>
                      {p.toFixed(1)}% asignado · objetivo {r.pct}% ({cop(r.target)})
                    </div>
                  </div>
                  <div className="w-40 sm:w-48 shrink-0">
                    {r.housing || r.car
                      ? <div className="num text-right text-lg font-bold">{cop(r.value)}</div>
                      : <MoneyInput label={r.label} value={r.value} onChange={v => setVal(r.id, v)} />}
                  </div>
                </div>
                <div className="mt-3 h-2.5 rounded-full overflow-hidden" style={{ background: "var(--track)" }} role="progressbar" aria-valuenow={Math.round(p)} aria-valuemin="0" aria-valuemax="100">
                  <div className="h-full rounded-full transition-all" style={{ width: Math.min(p / r.pct * 100, 100) + "%", background: over_ ? "var(--red)" : r.color }} />
                </div>

                {r.car && (
                  <button onClick={() => document.getElementById("carro").scrollIntoView({ behavior: "smooth" })}
                    className="mt-3 text-sm font-semibold" style={{ color: "#A67B5B" }}>
                    Ver desglose del carro abajo
                  </button>
                )}
                {r.housing && (
                  <div className="mt-3">
                    <button onClick={() => setOpen(!open)} className="flex items-center gap-1 text-sm font-semibold" style={{ color: "#A67B5B" }} aria-expanded={open}>
                      <span style={{ transform: open ? "rotate(180deg)" : "none", transition: ".15s", display: "inline-flex" }}><Icon n="chev" s={16} /></span>
                      {open ? "Ocultar desglose" : "Ver desglose"}
                    </button>
                    {open && (
                      <div className="grid gap-2 mt-2 sm:grid-cols-3">
                        {r.subs.map(s => (
                          <div key={s.id}>
                            <div className="text-sm mb-1" style={{ color: "var(--mute)" }}>{s.label}</div>
                            <MoneyInput label={s.label} value={s.value} onChange={v => setVal("vivienda." + s.id, v)} />
                          </div>
                        ))}
                      </div>
                    )}
                  </div>
                )}
              </article>
            );
          })}
        </section>

        {/* Dona */}
        <aside className="lg:col-span-2 card p-5 h-fit lg:sticky lg:top-4">
          <h2 className="text-xl font-bold mb-3">Para dónde va su plata</h2>
          <Donut slices={rows} base={Math.max(base, assigned)} active={active} setActive={setActive} />
          <ul className="mt-4 grid gap-1.5 text-sm">
            {rows.map(r => (
              <li key={r.id} className="flex items-center gap-2 cursor-pointer" onMouseEnter={() => setActive(r.id)} onMouseLeave={() => setActive(null)}>
                <span className="rounded-full" style={{ width: 10, height: 10, background: r.color }} />
                <span className="flex-1">{r.label}</span>
                <span className="font-semibold">{pctOf(r.value).toFixed(0)}%</span>
              </li>
            ))}
            {left > 0 && (
              <li className="flex items-center gap-2" style={{ color: "var(--mute)" }}>
                <span className="rounded-full" style={{ width: 10, height: 10, background: "var(--track)", border: "1px solid var(--line)" }} />
                <span className="flex-1">Sin asignar</span><span className="font-semibold">{pctOf(left).toFixed(0)}%</span>
              </li>
            )}
          </ul>
        </aside>
      </div>
      {/* Desglose del carro */}
      {(() => {
        const row = rows.find(r => r.car);
        if (!row) return null;
        const yr = Object.values(car).reduce((a, b) => a + b, 0);
        const over_ = row.value > row.target + 1;
        return (
          <section id="carro" className="card p-4 sm:p-5 mt-4">
            <div className="flex items-center gap-3 mb-1">
              <span className="grid place-items-center rounded-xl" style={{ width: 38, height: 38, background: row.color, color: "#FFF8EE" }}><Icon n="car" /></span>
              <h2 className="text-xl font-bold">Desglose del ahorro para el carro</h2>
            </div>
            <p className="text-sm mb-4" style={{ color: "var(--mute)" }}>
              Escriba lo que paga al año por cada cosa. La cuota mensual es ese valor dividido en 12, para llegar con la plata el día del pago.
            </p>
            <div className="grid gap-3 sm:grid-cols-3">
              {CAR_ITEMS.map(i => (
                <div key={i.id} className="rounded-xl p-3" style={{ background: "var(--bg)", border: "1px solid var(--line)" }}>
                  <div className="text-sm font-semibold mb-2">{i.label}</div>
                  <MoneyInput label={i.label + " por año"} value={car[i.id]} onChange={v => setCar(c => ({ ...c, [i.id]: v }))} />
                  <div className="text-sm mt-2" style={{ color: "var(--mute)" }}>Aparte cada mes: <b style={{ color: "var(--ink)" }}>{cop(car[i.id] / 12)}</b></div>
                </div>
              ))}
            </div>
            <div className="flex flex-wrap gap-x-8 gap-y-1 mt-4 text-sm">
              <span>Total al año: <b>{cop(yr)}</b></span>
              <span>Cuota mensual: <b className="num text-base">{cop(row.value)}</b></span>
              <span style={{ color: over_ ? "var(--red)" : "var(--mute)" }}>
                Objetivo del {row.pct}%: {cop(row.target)} {over_ ? "· la cuota lo supera" : "· dentro del objetivo"}
              </span>
            </div>
            <p className="text-xs mt-3" style={{ color: "var(--mute)" }}>Los valores iniciales son de ejemplo. Cámbielos por lo que realmente le cobran.</p>
          </section>
        );
      })()}
      {/* Efectivo vs cuenta de ahorro */}
      {(() => {
        const tot = cash + bank, pc = tot ? cash / tot * 100 : 0;
        return (
          <section className="card p-4 sm:p-5 mt-4">
            <div className="flex items-center gap-3 mb-1">
              <span className="grid place-items-center rounded-xl" style={{ width: 38, height: 38, background: "#A67B5B", color: "#FFF8EE" }}><Icon n="wallet" /></span>
              <h2 className="text-xl font-bold">Dónde está su plata hoy</h2>
            </div>
            <p className="text-sm mb-4" style={{ color: "var(--mute)" }}>Escriba cuánto tiene ahora en efectivo y cuánto en la cuenta de ahorro. Es una foto de hoy, no cambia el presupuesto.</p>
            <div className="grid gap-3 sm:grid-cols-2">
              <div>
                <div className="text-sm font-semibold mb-2">En efectivo</div>
                <MoneyInput big label="Dinero en efectivo" value={cash} onChange={setCash} />
              </div>
              <div>
                <div className="text-sm font-semibold mb-2">En la cuenta de ahorro</div>
                <MoneyInput big label="Dinero en la cuenta de ahorro" value={bank} onChange={setBank} />
              </div>
            </div>
            <div className="mt-4 flex h-3 rounded-full overflow-hidden" style={{ background: "var(--track)" }} role="img" aria-label="Proporción entre efectivo y cuenta de ahorro">
              <div style={{ width: pc + "%", background: "#E8A08A", transition: "width .2s" }} />
              <div style={{ width: (tot ? 100 - pc : 0) + "%", background: "#8C6A4F", transition: "width .2s" }} />
            </div>
            <div className="flex flex-wrap gap-x-8 gap-y-1 mt-3 text-sm">
              <span><span className="inline-block rounded-full mr-1.5" style={{ width: 10, height: 10, background: "#E8A08A" }} />Efectivo: <b>{pc.toFixed(0)}%</b></span>
              <span><span className="inline-block rounded-full mr-1.5" style={{ width: 10, height: 10, background: "#8C6A4F" }} />Cuenta de ahorro: <b>{tot ? (100 - pc).toFixed(0) : 0}%</b></span>
              <span>Total disponible: <b className="num text-base">{cop(tot)}</b></span>
            </div>
          </section>
        );
      })()}
      <p className="text-xs mt-6" style={{ color: "var(--mute)" }}>Herramienta de apoyo: no es asesoría financiera. Los valores se calculan en su navegador y no se guardan.</p>
    </main>
  );
}
ReactDOM.createRoot(document.getElementById("root")).render(<App />);
</script>
</body>
</html>
