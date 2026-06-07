import { useState, useEffect, useCallback } from "react";

var STORE = "vida_app_v1";
function load() {
  try { var r = localStorage.getItem(STORE); if (r) return JSON.parse(r); } catch(e) {}
  return { pillars: [], darkMode: true };
}
function save(data) {
  try { localStorage.setItem(STORE, JSON.stringify(data)); } catch(e) {}
}
function uid() { return Date.now().toString(36) + Math.random().toString(36).slice(2); }
function todayStr() {
  var d = new Date();
  return d.getFullYear() + "-" + String(d.getMonth()+1).padStart(2,"0") + "-" + String(d.getDate()).padStart(2,"0");
}
function formatDate(str) {
  if (!str) return "";
  var parts = str.split("-");
  var months = ["Ene","Feb","Mar","Abr","May","Jun","Jul","Ago","Sep","Oct","Nov","Dic"];
  return parseInt(parts[2]) + " " + months[parseInt(parts[1])-1] + " " + parts[0];
}
function clamp(v,min,max) { return Math.max(min, Math.min(max, v)); }

function calcObjectiveScore(obj) {
  if (!obj.metrics || obj.metrics.length === 0) return null;
  var scores = obj.metrics.map(function(m) {
    if (m.type === "count") {
      var total = Object.values(m.entries || {}).reduce(function(a,b){return a+(b||0);},0);
      return m.target > 0 ? clamp(Math.round((total / m.target) * 100), 0, 100) : null;
    } else if (m.type === "check") {
      var entries = Object.values(m.entries || {});
      if (entries.length === 0) return null;
      return Math.round((entries.filter(function(v){return v;}).length / entries.length) * 100);
    } else if (m.type === "scale") {
      var vals = Object.values(m.entries || {}).filter(function(v){return v > 0;});
      if (vals.length === 0) return null;
      return Math.round((vals.reduce(function(a,b){return a+b;},0) / vals.length / 5) * 100);
    }
    return null;
  }).filter(function(s){return s!==null;});
  if (scores.length === 0) return null;
  return Math.round(scores.reduce(function(a,b){return a+b;},0) / scores.length);
}

function calcPillarScore(pillar) {
  if (!pillar.objectives || pillar.objectives.length === 0) return null;
  var active = pillar.objectives.filter(function(o) { return o.active !== false; });
  if (active.length === 0) return null;
  var scores = active.map(function(obj) { return calcObjectiveScore(obj); }).filter(function(s) { return s !== null; });
  if (scores.length === 0) return null;
  return Math.round(scores.reduce(function(a,b){return a+b;},0) / scores.length);
}

function calcGlobalScore(pillars) {
  var active = pillars.filter(function(p){return p.objectives && p.objectives.some(function(o){return o.active!==false;});});
  if (active.length === 0) return null;
  var weighted = active.map(function(p) {
    var score = calcPillarScore(p);
    if (score === null) return null;
    return { score: score, weight: p.weight || 0 };
  }).filter(function(x){return x!==null;});
  if (weighted.length === 0) return null;
  var wTotal = weighted.reduce(function(a,x){return a+x.weight;},0);
  if (wTotal === 0) return null;
  return Math.round(weighted.reduce(function(a,x){return a+x.score*x.weight;},0) / wTotal);
}

var PILLAR_COLORS = ["#EF4444","#F97316","#EAB308","#22C55E","#06B6D4","#3B82F6","#8B5CF6","#EC4899","#14B8A6","#F59E0B"];

function exportToCSV(pillars) {
  var rows = [["PILAR","PESO %","OBJETIVO","PERIODO","METRICA","TIPO","META","FECHA","VALOR","SCORE OBJ","SCORE PILAR","SCORE GLOBAL"]];
  var globalScore = calcGlobalScore(pillars);
  pillars.forEach(function(p) {
    var pillarScore = calcPillarScore(p);
    (p.objectives || []).forEach(function(obj) {
      var objScore = calcObjectiveScore(obj);
      (obj.metrics || []).forEach(function(m) {
        var entries = Object.entries(m.entries || {});
        if (entries.length === 0) {
          rows.push([p.name, p.weight||0, obj.name, obj.period||"", m.name, m.type, m.target||"", "", "", objScore||"", pillarScore||"", globalScore||""]);
        } else {
          entries.forEach(function(en, i) {
            rows.push([p.name, p.weight||0, obj.name, obj.period||"", m.name, m.type, m.target||"", en[0], en[1], i===0?(objScore||""):"", i===0?(pillarScore||""):"", i===0?(globalScore||""):""]);
          });
        }
      });
    });
  });
  var csv = rows.map(function(r){return r.map(function(c){return '"'+(String(c||"").replace(/"/g,'""'))+'"';}).join(",");}).join("\n");
  var blob = new Blob([csv], {type:"text/csv;charset=utf-8;"});
  var url = URL.createObjectURL(blob);
  var a = document.createElement("a");
  a.href = url; a.download = "vida_tracker_" + todayStr() + ".csv";
  document.body.appendChild(a); a.click(); document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

var darkTheme = {
  bg: "#0A0A0A", surface: "#141414", surface2: "#1E1E1E", surface3: "#2A2A2A",
  border: "rgba(255,255,255,0.08)", border2: "rgba(255,255,255,0.14)",
  text: "#F5F5F5", text2: "rgba(255,255,255,0.55)", text3: "rgba(255,255,255,0.3)",
  green: "#22C55E", red: "#EF4444", accent: "#F5F5F5", accentText: "#000",
};
var lightTheme = {
  bg: "#FAFAFA", surface: "#FFFFFF", surface2: "#F3F3F3", surface3: "#E8E8E8",
  border: "rgba(0,0,0,0.08)", border2: "rgba(0,0,0,0.14)",
  text: "#111111", text2: "rgba(0,0,0,0.5)", text3: "rgba(0,0,0,0.28)",
  green: "#16A34A", red: "#DC2626", accent: "#111111", accentText: "#fff",
};

function ScoreRing(props) {
  var score = props.score; var size = props.size || 80; var t = props.t;
  var stroke = size < 60 ? 5 : 7;
  var r = (size - stroke) / 2;
  var circ = 2 * Math.PI * r;
  var pct = score !== null ? clamp(score, 0, 100) / 100 : 0;
  var c = score === null ? t.border2 : score >= 70 ? t.green : score >= 40 ? "#F97316" : t.red;
  return (
    <svg width={size} height={size} style={{transform:"rotate(-90deg)"}}>
      <circle cx={size/2} cy={size/2} r={r} fill="none" stroke={t.border} strokeWidth={stroke}/>
      <circle cx={size/2} cy={size/2} r={r} fill="none" stroke={c} strokeWidth={stroke}
        strokeLinecap="round" strokeDasharray={circ} strokeDashoffset={circ - pct*circ}
        style={{transition:"stroke-dashoffset 0.8s cubic-bezier(0.34,1.56,0.64,1)"}}/>
    </svg>
  );
}

function Modal(props) {
  var t = props.t;
  return (
    <div style={{position:"fixed",inset:0,zIndex:200,background:"rgba(0,0,0,0.7)",backdropFilter:"blur(8px)",display:"flex",alignItems:"flex-end",justifyContent:"center",animation:"fadeIn 0.15s ease"}} onClick={props.onClose}>
      <div style={{width:"100%",maxWidth:520,background:t.surface,borderRadius:"20px 20px 0 0",padding:"0 0 40px",maxHeight:"90dvh",overflowY:"auto",animation:"slideUp 0.3s cubic-bezier(0.34,1.56,0.64,1)"}} onClick={function(e){e.stopPropagation();}}>
        <div style={{position:"sticky",top:0,background:t.surface,padding:"14px 20px 10px",borderBottom:"1px solid "+t.border,zIndex:1}}>
          <div style={{width:36,height:4,borderRadius:2,background:t.border2,margin:"0 auto 12px"}}/>
          <div style={{display:"flex",justifyContent:"space-between",alignItems:"center"}}>
            <span style={{fontFamily:"'Playfair Display',Georgia,serif",fontSize:18,fontWeight:700,color:t.text}}>{props.title}</span>
            <button onClick={props.onClose} style={{background:"none",border:"none",cursor:"pointer",color:t.text3,fontSize:22,lineHeight:1,padding:"4px 8px",fontFamily:"sans-serif"}}>x</button>
          </div>
        </div>
        <div style={{padding:"16px 20px"}}>{props.children}</div>
      </div>
    </div>
  );
}

function FInput(props) {
  var t = props.t;
  return (
    <div style={{marginBottom:14}}>
      {props.label && <label style={{display:"block",fontSize:11,fontWeight:700,color:t.text2,marginBottom:6,letterSpacing:0.5,fontFamily:"'DM Sans',sans-serif"}}>{props.label.toUpperCase()}</label>}
      <input type={props.type||"text"} value={props.value} onChange={props.onChange} placeholder={props.placeholder}
        style={{width:"100%",padding:"10px 12px",borderRadius:10,background:t.surface2,border:"1px solid "+t.border2,color:t.text,fontSize:14,outline:"none",fontFamily:"'DM Sans',sans-serif",boxSizing:"border-box"}}/>
    </div>
  );
}

function Btn(props) {
  var t = props.t;
  var bg = props.variant==="ghost"?"transparent":props.variant==="danger"?t.red:props.color||t.accent;
  var cl = props.variant==="ghost"?t.text2:props.variant==="danger"?"#fff":t.accentText;
  if (props.color) cl = "#fff";
  return (
    <button onClick={props.onClick} style={{padding:props.small?"7px 14px":"11px 20px",borderRadius:10,border:props.variant==="ghost"?"1px solid "+t.border2:"none",background:bg,color:cl,cursor:"pointer",fontSize:props.small?13:14,fontWeight:600,fontFamily:"'DM Sans',sans-serif",transition:"opacity 0.15s ease"}}
      onMouseEnter={function(e){e.currentTarget.style.opacity="0.8";}}
      onMouseLeave={function(e){e.currentTarget.style.opacity="1";}}
    >{props.children}</button>
  );
}

function Badge(props) {
  return <span style={{display:"inline-block",padding:"2px 8px",borderRadius:20,background:props.color+"22",color:props.color,fontSize:11,fontWeight:600,letterSpacing:0.3,fontFamily:"'DM Sans',sans-serif"}}>{props.label}</span>;
}

var PERIODS = ["Diario","Semanal","Mensual","Trimestral","Semestral","Anual"];

function PillarForm(props) {
  var pillar = props.pillar; var t = props.t;
  var [name, setName] = useState(pillar ? pillar.name : "");
  var [weight, setWeight] = useState(pillar ? String(pillar.weight) : "10");
  var [color, setColor] = useState(pillar ? pillar.color : PILLAR_COLORS[0]);
  var [desc, setDesc] = useState(pillar ? (pillar.desc||"") : "");
  function submit() {
    if (!name.trim()) return;
    props.onSave({ name:name.trim(), weight:parseInt(weight)||10, color, desc });
  }
  return (
    <div>
      <FInput label="Nombre del pilar" value={name} onChange={function(e){setName(e.target.value);}} placeholder="Ej: Cuerpo y energia" t={t}/>
      <FInput label="Descripcion (opcional)" value={desc} onChange={function(e){setDesc(e.target.value);}} placeholder="De que trata este pilar..." t={t}/>
      <FInput label="Peso (%)" value={weight} onChange={function(e){setWeight(e.target.value);}} type="number" placeholder="20" t={t}/>
      <div style={{marginBottom:16}}>
        <label style={{display:"block",fontSize:11,fontWeight:700,color:t.text2,marginBottom:8,letterSpacing:0.5,fontFamily:"'DM Sans',sans-serif"}}>COLOR DEL PILAR</label>
        <div style={{display:"flex",gap:8,flexWrap:"wrap"}}>
          {PILLAR_COLORS.map(function(c){return <button key={c} onClick={function(){setColor(c);}} style={{width:28,height:28,borderRadius:"50%",background:c,border:"none",cursor:"pointer",outline:color===c?"3px solid white":"2px solid transparent",outlineOffset:2,transition:"outline 0.15s"}}/>;})}
        </div>
      </div>
      <div style={{display:"flex",gap:10,justifyContent:"flex-end"}}>
        <Btn variant="ghost" onClick={props.onClose} t={t}>Cancelar</Btn>
        <Btn onClick={submit} color={color} t={t}>Guardar</Btn>
      </div>
    </div>
  );
}

function ObjectiveForm(props) {
  var obj = props.obj; var t = props.t;
  var [name, setName] = useState(obj ? obj.name : "");
  var [desc, setDesc] = useState(obj ? (obj.desc||"") : "");
  var [period, setPeriod] = useState(obj ? (obj.period||"Mensual") : "Mensual");
  var [metrics, setMetrics] = useState(obj ? JSON.parse(JSON.stringify(obj.metrics)) : []);
  var [mName, setMName] = useState("");
  var [mType, setMType] = useState("count");
  var [mTarget, setMTarget] = useState("");

  function addMetric() {
    if (!mName.trim()) return;
    var m = { id:uid(), name:mName.trim(), type:mType, entries:{} };
    if (mType === "count") m.target = parseInt(mTarget)||1;
    setMetrics(function(prev){return prev.concat([m]);});
    setMName(""); setMTarget("");
  }
  function removeMetric(id) { setMetrics(function(prev){return prev.filter(function(m){return m.id!==id;});}); }
  function submit() {
    if (!name.trim()) return;
    props.onSave({ name:name.trim(), desc, period, metrics, active:true });
  }
  return (
    <div>
      <FInput label="Nombre del objetivo" value={name} onChange={function(e){setName(e.target.value);}} placeholder="Ej: Ser mejor novio" t={t}/>
      <FInput label="Descripcion (opcional)" value={desc} onChange={function(e){setDesc(e.target.value);}} placeholder="Por que es importante..." t={t}/>
      <div style={{marginBottom:14}}>
        <label style={{display:"block",fontSize:11,fontWeight:700,color:t.text2,marginBottom:8,letterSpacing:0.5,fontFamily:"'DM Sans',sans-serif"}}>PERIODO</label>
        <div style={{display:"flex",gap:6,flexWrap:"wrap"}}>
          {PERIODS.map(function(p){return <button key={p} onClick={function(){setPeriod(p);}} style={{padding:"6px 12px",borderRadius:8,border:"1px solid "+t.border2,cursor:"pointer",background:period===p?t.accent:"transparent",color:period===p?t.accentText:t.text2,fontSize:12,fontWeight:500,fontFamily:"'DM Sans',sans-serif",transition:"all 0.15s"}}>{p}</button>;})}
        </div>
      </div>
      <div style={{marginBottom:16}}>
        <label style={{display:"block",fontSize:11,fontWeight:700,color:t.text2,marginBottom:8,letterSpacing:0.5,fontFamily:"'DM Sans',sans-serif"}}>METRICAS</label>
        {metrics.map(function(m){
          return (
            <div key={m.id} style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"8px 12px",borderRadius:8,background:t.surface2,marginBottom:6,border:"1px solid "+t.border}}>
              <div>
                <span style={{color:t.text,fontSize:13,fontWeight:500,fontFamily:"'DM Sans',sans-serif"}}>{m.name}</span>
                <span style={{color:t.text3,fontSize:11,marginLeft:8,fontFamily:"'DM Sans',sans-serif"}}>{m.type==="count"?"Contador (meta: "+m.target+")":m.type==="check"?"Check diario":"Escala 1-5"}</span>
              </div>
              <button onClick={function(){removeMetric(m.id);}} style={{background:"none",border:"none",cursor:"pointer",color:t.red,fontSize:16,padding:"2px 6px"}}>x</button>
            </div>
          );
        })}
        <div style={{background:t.surface2,borderRadius:10,padding:12,border:"1px solid "+t.border}}>
          <input value={mName} onChange={function(e){setMName(e.target.value);}} placeholder="Nombre de la metrica" style={{width:"100%",padding:"8px 10px",borderRadius:8,background:t.surface3,border:"1px solid "+t.border2,color:t.text,fontSize:13,outline:"none",fontFamily:"'DM Sans',sans-serif",boxSizing:"border-box",marginBottom:8}}/>
          <div style={{display:"flex",gap:6,marginBottom:8}}>
            {[["count","Contador"],["check","Check"],["scale","Escala 1-5"]].map(function(opt){return <button key={opt[0]} onClick={function(){setMType(opt[0]);}} style={{flex:1,padding:"6px 4px",borderRadius:8,border:"1px solid "+t.border2,cursor:"pointer",background:mType===opt[0]?t.accent:"transparent",color:mType===opt[0]?t.accentText:t.text2,fontSize:11,fontWeight:500,fontFamily:"'DM Sans',sans-serif"}}>{opt[1]}</button>;})}
          </div>
          {mType==="count" && <input value={mTarget} onChange={function(e){setMTarget(e.target.value);}} placeholder="Meta numerica (ej: 8)" type="number" style={{width:"100%",padding:"8px 10px",borderRadius:8,background:t.surface3,border:"1px solid "+t.border2,color:t.text,fontSize:13,outline:"none",fontFamily:"'DM Sans',sans-serif",boxSizing:"border-box",marginBottom:8}}/>}
          <button onClick={addMetric} style={{width:"100%",padding:"8px",borderRadius:8,border:"1px dashed "+t.border2,background:"transparent",color:t.text2,cursor:"pointer",fontSize:13,fontFamily:"'DM Sans',sans-serif"}}>+ Agregar metrica</button>
        </div>
      </div>
      <div style={{display:"flex",gap:10,justifyContent:"flex-end"}}>
        <Btn variant="ghost" onClick={props.onClose} t={t}>Cancelar</Btn>
        <Btn onClick={submit} t={t}>Guardar objetivo</Btn>
      </div>
    </div>
  );
}

function EntryModal(props) {
  var metric = props.metric; var t = props.t;
  var [selDate, setSelDate] = useState(todayStr());
  var [value, setValue] = useState(metric.entries[todayStr()] !== undefined ? String(metric.entries[todayStr()]) : "");
  var [viewMonth, setViewMonth] = useState(new Date());

  var year = viewMonth.getFullYear();
  var month = viewMonth.getMonth();
  var daysInMonth = new Date(year,month+1,0).getDate();
  var firstDay = (new Date(year,month,1).getDay()+6)%7;
  var monthNames = ["Enero","Febrero","Marzo","Abril","Mayo","Junio","Julio","Agosto","Septiembre","Octubre","Noviembre","Diciembre"];
  var dayNames = ["Lu","Ma","Mi","Ju","Vi","Sa","Do"];

  function selectDay(d) {
    var k = year+"-"+String(month+1).padStart(2,"0")+"-"+String(d).padStart(2,"0");
    setSelDate(k);
    setValue(metric.entries[k] !== undefined ? String(metric.entries[k]) : "");
  }
  function submit() {
    var v;
    if (metric.type === "check") v = value === "true";
    else if (metric.type === "scale") v = parseInt(value)||1;
    else v = parseInt(value)||0;
    props.onSave(selDate, v);
    props.onClose();
  }
  function removeEntry() { props.onSave(selDate, null); props.onClose(); }

  var cells = [];
  for (var i=0;i<firstDay;i++) cells.push(null);
  for (var dd=1;dd<=daysInMonth;dd++) cells.push(dd);

  return (
    <Modal title={props.objName + " - " + metric.name} onClose={props.onClose} t={t}>
      <div style={{marginBottom:16}}>
        <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:10}}>
          <button onClick={function(){setViewMonth(new Date(year,month-1,1));}} style={{background:"none",border:"none",cursor:"pointer",color:t.text2,fontSize:20,padding:"4px 10px"}}>{"<"}</button>
          <span style={{fontFamily:"'Playfair Display',Georgia,serif",fontSize:15,fontWeight:700,color:t.text}}>{monthNames[month]} {year}</span>
          <button onClick={function(){setViewMonth(new Date(year,month+1,1));}} style={{background:"none",border:"none",cursor:"pointer",color:t.text2,fontSize:20,padding:"4px 10px"}}>{">"}</button>
        </div>
        <div style={{display:"grid",gridTemplateColumns:"repeat(7,1fr)",gap:3,marginBottom:4}}>
          {dayNames.map(function(dn){return <div key={dn} style={{textAlign:"center",fontSize:10,color:t.text3,padding:"2px 0",fontFamily:"'DM Sans',sans-serif"}}>{dn}</div>;})}
        </div>
        <div style={{display:"grid",gridTemplateColumns:"repeat(7,1fr)",gap:3}}>
          {cells.map(function(d,i){
            if (!d) return <div key={"e"+i}/>;
            var k = year+"-"+String(month+1).padStart(2,"0")+"-"+String(d).padStart(2,"0");
            var hasEntry = metric.entries[k] !== undefined;
            var isSel = selDate === k;
            var isToday = k === todayStr();
            return <button key={d} onClick={function(){selectDay(d);}} style={{aspectRatio:"1",borderRadius:8,border:"none",cursor:"pointer",background:isSel?t.accent:hasEntry?t.green+"33":isToday?t.border2:"transparent",color:isSel?t.accentText:hasEntry?t.green:t.text,fontSize:12,fontWeight:isSel||isToday?700:400,fontFamily:"'DM Sans',sans-serif",transition:"all 0.1s"}}>{d}</button>;
          })}
        </div>
      </div>
      <div style={{background:t.surface2,borderRadius:12,padding:14,marginBottom:14}}>
        <p style={{fontSize:11,color:t.text2,margin:"0 0 10px",fontWeight:700,letterSpacing:0.5,fontFamily:"'DM Sans',sans-serif"}}>REGISTRO - {formatDate(selDate)}</p>
        {metric.type === "check" && (
          <div style={{display:"flex",gap:10}}>
            <button onClick={function(){setValue("true");}} style={{flex:1,padding:"10px",borderRadius:10,border:"1px solid "+t.border2,cursor:"pointer",background:value==="true"?t.green:"transparent",color:value==="true"?"#fff":t.text2,fontFamily:"'DM Sans',sans-serif",fontSize:14,fontWeight:600}}>Hecho</button>
            <button onClick={function(){setValue("false");}} style={{flex:1,padding:"10px",borderRadius:10,border:"1px solid "+t.border2,cursor:"pointer",background:value==="false"?t.red:"transparent",color:value==="false"?"#fff":t.text2,fontFamily:"'DM Sans',sans-serif",fontSize:14,fontWeight:600}}>No hecho</button>
          </div>
        )}
        {metric.type === "scale" && (
          <div style={{display:"flex",gap:8}}>
            {[1,2,3,4,5].map(function(n){return <button key={n} onClick={function(){setValue(String(n));}} style={{flex:1,padding:"10px 0",borderRadius:10,border:"1px solid "+t.border2,cursor:"pointer",background:String(n)===String(value)?t.accent:"transparent",color:String(n)===String(value)?t.accentText:t.text2,fontFamily:"'DM Sans',sans-serif",fontSize:16,fontWeight:700}}>{n}</button>;})}
          </div>
        )}
        {metric.type === "count" && (
          <div style={{display:"flex",alignItems:"center",gap:10}}>
            <button onClick={function(){setValue(function(v){return String(Math.max(0,(parseInt(v)||0)-1));});}} style={{width:40,height:40,borderRadius:10,border:"1px solid "+t.border2,background:"transparent",color:t.text,fontSize:20,cursor:"pointer",fontFamily:"'DM Sans',sans-serif"}}>-</button>
            <input type="number" value={value} onChange={function(e){setValue(e.target.value);}} style={{flex:1,padding:"10px",borderRadius:10,border:"1px solid "+t.border2,background:t.surface3,color:t.text,fontSize:20,fontWeight:700,textAlign:"center",outline:"none",fontFamily:"'DM Sans',sans-serif"}}/>
            <button onClick={function(){setValue(function(v){return String((parseInt(v)||0)+1);});}} style={{width:40,height:40,borderRadius:10,border:"1px solid "+t.border2,background:"transparent",color:t.text,fontSize:20,cursor:"pointer",fontFamily:"'DM Sans',sans-serif"}}>+</button>
          </div>
        )}
        {metric.entries[selDate] !== undefined && <button onClick={removeEntry} style={{marginTop:10,width:"100%",padding:"7px",borderRadius:8,border:"1px solid "+t.red+"44",background:"transparent",color:t.red,cursor:"pointer",fontSize:12,fontFamily:"'DM Sans',sans-serif"}}>Eliminar registro de este dia</button>}
      </div>
      <div style={{display:"flex",gap:10,justifyContent:"flex-end"}}>
        <Btn variant="ghost" onClick={props.onClose} t={t}>Cancelar</Btn>
        <Btn onClick={submit} t={t}>Guardar</Btn>
      </div>
    </Modal>
  );
}

function PillarScreen(props) {
  var pillar = props.pillar; var t = props.t;
  var [showAddObj, setShowAddObj] = useState(false);
  var [editObj, setEditObj] = useState(null);
  var [entryInfo, setEntryInfo] = useState(null);

  function addObjective(data) {
    var newObj = Object.assign({ id:uid() }, data);
    props.onUpdate(Object.assign({},pillar,{objectives:(pillar.objectives||[]).concat([newObj])}));
    setShowAddObj(false);
  }
  function updateObjective(id, data) {
    props.onUpdate(Object.assign({},pillar,{objectives:(pillar.objectives||[]).map(function(o){return o.id===id?Object.assign({},o,data):o;})}));
    setEditObj(null);
  }
  function deleteObjective(id) {
    if (!window.confirm("Eliminar objetivo?")) return;
    props.onUpdate(Object.assign({},pillar,{objectives:(pillar.objectives||[]).filter(function(o){return o.id!==id;})}));
  }
  function saveEntry(objId, metricId, date, value) {
    props.onUpdate(Object.assign({},pillar,{
      objectives:(pillar.objectives||[]).map(function(o){
        if (o.id!==objId) return o;
        return Object.assign({},o,{metrics:o.metrics.map(function(m){
          if (m.id!==metricId) return m;
          var ne = Object.assign({},m.entries);
          if (value===null) delete ne[date]; else ne[date]=value;
          return Object.assign({},m,{entries:ne});
        })});
      })
    }));
    setEntryInfo(null);
  }

  var score = calcPillarScore(pillar);
  var scoreColor = score===null?t.text3:score>=70?t.green:score>=40?"#F97316":t.red;

  return (
    <div style={{minHeight:"100dvh",background:t.bg,paddingBottom:80}}>
      <div style={{background:t.surface,borderBottom:"1px solid "+t.border,padding:"max(env(safe-area-inset-top,44px),44px) 20px 16px",position:"sticky",top:0,zIndex:10}}>
        <button onClick={props.onBack} style={{background:"none",border:"none",cursor:"pointer",color:t.text2,fontSize:13,fontFamily:"'DM Sans',sans-serif",marginBottom:10,padding:"4px 0",display:"flex",alignItems:"center",gap:4,fontWeight:500}}>{"< Volver"}</button>
        <div style={{display:"flex",alignItems:"flex-start",justifyContent:"space-between",gap:8}}>
          <div style={{flex:1}}>
            <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:4}}>
              <div style={{width:10,height:10,borderRadius:"50%",background:pillar.color,flexShrink:0,marginTop:2}}/>
              <span style={{fontFamily:"'Playfair Display',Georgia,serif",fontSize:22,fontWeight:700,color:t.text}}>{pillar.name}</span>
            </div>
            {pillar.desc && <p style={{fontSize:12,color:t.text2,margin:"0 0 0 20px",fontFamily:"'DM Sans',sans-serif"}}>{pillar.desc}</p>}
          </div>
          <div style={{textAlign:"right",flexShrink:0}}>
            <div style={{fontSize:28,fontWeight:800,color:scoreColor,fontFamily:"'Playfair Display',Georgia,serif",lineHeight:1}}>{score!==null?score+"%":"--"}</div>
            <div style={{fontSize:10,color:t.text3,fontWeight:600,letterSpacing:0.5,fontFamily:"'DM Sans',sans-serif"}}>PESO: {pillar.weight}%</div>
          </div>
        </div>
      </div>
      <div style={{padding:"16px 20px"}}>
        {(pillar.objectives||[]).length===0 && (
          <div style={{textAlign:"center",padding:"60px 0",color:t.text3,fontFamily:"'DM Sans',sans-serif"}}>
            <p style={{fontSize:36,marginBottom:12}}>?#127919;</p>
            <p style={{fontSize:15,color:t.text2}}>Sin objetivos en este pilar</p>
          </div>
        )}
        {(pillar.objectives||[]).map(function(obj){
          var objScore = calcObjectiveScore(obj);
          var objColor = objScore===null?t.text3:objScore>=70?t.green:objScore>=40?"#F97316":t.red;
          return (
            <div key={obj.id} style={{background:t.surface,borderRadius:14,marginBottom:12,border:"1px solid "+t.border,overflow:"hidden"}}>
              <div style={{padding:"14px 16px",borderBottom:"1px solid "+t.border}}>
                <div style={{display:"flex",alignItems:"flex-start",justifyContent:"space-between",gap:8}}>
                  <div style={{flex:1}}>
                    <div style={{display:"flex",alignItems:"center",gap:8,flexWrap:"wrap",marginBottom:4}}>
                      <span style={{fontFamily:"'Playfair Display',Georgia,serif",fontSize:15,fontWeight:700,color:t.text}}>{obj.name}</span>
                      <Badge label={obj.period} color={pillar.color}/>
                    </div>
                    {obj.desc && <p style={{fontSize:12,color:t.text2,margin:0,fontFamily:"'DM Sans',sans-serif"}}>{obj.desc}</p>}
                  </div>
                  <div style={{display:"flex",alignItems:"center",gap:6,flexShrink:0}}>
                    <span style={{fontSize:20,fontWeight:800,color:objColor,fontFamily:"'Playfair Display',Georgia,serif"}}>{objScore!==null?objScore+"%":"--"}</span>
                    <button onClick={function(){setEditObj(obj);}} style={{background:"none",border:"none",cursor:"pointer",color:t.text3,fontSize:15,padding:"4px"}}>?#9998;</button>
                    <button onClick={function(){deleteObjective(obj.id);}} style={{background:"none",border:"none",cursor:"pointer",color:t.red+"88",fontSize:14,padding:"4px"}}>x</button>
                  </div>
                </div>
              </div>
              <div style={{padding:"10px 16px 14px"}}>
                {(obj.metrics||[]).length===0 && <p style={{fontSize:12,color:t.text3,fontFamily:"'DM Sans',sans-serif",margin:0}}>Sin metricas</p>}
                {(obj.metrics||[]).map(function(m){
                  var todayEntry = m.entries[todayStr()];
                  var totalCount = m.type==="count" ? Object.values(m.entries||{}).reduce(function(a,b){return a+(b||0);},0) : null;
                  return (
                    <div key={m.id} style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"9px 10px",borderRadius:8,background:t.surface2,marginBottom:6,cursor:"pointer",transition:"opacity 0.1s"}}
                      onClick={function(){setEntryInfo({metric:m,objId:obj.id,objName:obj.name});}}
                      onMouseEnter={function(e){e.currentTarget.style.opacity="0.75";}}
                      onMouseLeave={function(e){e.currentTarget.style.opacity="1";}}
                    >
                      <div>
                        <span style={{fontSize:13,fontWeight:500,color:t.text,fontFamily:"'DM Sans',sans-serif"}}>{m.name}</span>
                        <span style={{fontSize:11,color:t.text3,marginLeft:8,fontFamily:"'DM Sans',sans-serif"}}>
                          {m.type==="count"?"Total: "+(totalCount||0)+(m.target?" / "+m.target:""):m.type==="check"?"Diario":"Escala 1-5"}
                        </span>
                      </div>
                      <div style={{display:"flex",alignItems:"center",gap:8}}>
                        {todayEntry!==undefined && (
                          <span style={{fontSize:12,fontWeight:600,padding:"3px 8px",borderRadius:6,background:t.green+"22",color:t.green,fontFamily:"'DM Sans',sans-serif"}}>
                            {m.type==="check"?(todayEntry?"Hecho":"No"):m.type==="scale"?todayEntry+"/5":todayEntry}
                          </span>
                        )}
                        <span style={{color:t.text3,fontSize:14}}>{">"}</span>
                      </div>
                    </div>
                  );
                })}
              </div>
            </div>
          );
        })}
        <button onClick={function(){setShowAddObj(true);}} style={{width:"100%",padding:"14px",borderRadius:14,border:"1px dashed "+t.border2,background:"transparent",color:t.text2,cursor:"pointer",fontSize:14,fontFamily:"'DM Sans',sans-serif",fontWeight:500}}>+ Agregar objetivo</button>
      </div>
      {showAddObj && <Modal title="Nuevo objetivo" onClose={function(){setShowAddObj(false);}} t={t}><ObjectiveForm onSave={addObjective} onClose={function(){setShowAddObj(false);}} t={t}/></Modal>}
      {editObj && <Modal title="Editar objetivo" onClose={function(){setEditObj(null);}} t={t}><ObjectiveForm obj={editObj} onSave={function(data){updateObjective(editObj.id,data);}} onClose={function(){setEditObj(null);}} t={t}/></Modal>}
      {entryInfo && <EntryModal metric={entryInfo.metric} objName={entryInfo.objName} onSave={function(date,val){saveEntry(entryInfo.objId,entryInfo.metric.id,date,val);}} onClose={function(){setEntryInfo(null);}} t={t}/>}
    </div>
  );
}

function HomeScreen(props) {
  var data = props.data; var t = props.t;
  var [showAddPillar, setShowAddPillar] = useState(false);
  var [editPillar, setEditPillar] = useState(null);
  var [activePillarId, setActivePillarId] = useState(null);

  var globalScore = calcGlobalScore(data.pillars);
  var totalWeight = data.pillars.reduce(function(a,p){return a+(p.weight||0);},0);
  var scoreColor = globalScore===null?t.text3:globalScore>=70?t.green:globalScore>=40?"#F97316":t.red;

  function addPillar(pd) {
    props.onUpdateData(Object.assign({},data,{pillars:data.pillars.concat([Object.assign({id:uid(),objectives:[]},pd)])}));
    setShowAddPillar(false);
  }
  function updatePillar(id, pd) {
    props.onUpdateData(Object.assign({},data,{pillars:data.pillars.map(function(p){return p.id===id?Object.assign({},p,pd):p;})}));
    setEditPillar(null);
  }
  function deletePillar(id) {
    if (!window.confirm("Eliminar este pilar y todos sus objetivos?")) return;
    props.onUpdateData(Object.assign({},data,{pillars:data.pillars.filter(function(p){return p.id!==id;})}));
  }
  function updatePillarFull(updated) {
    props.onUpdateData(Object.assign({},data,{pillars:data.pillars.map(function(p){return p.id===updated.id?updated:p;})}));
  }

  if (activePillarId) {
    var ap = data.pillars.find(function(p){return p.id===activePillarId;});
    if (ap) return <PillarScreen pillar={ap} onBack={function(){setActivePillarId(null);}} onUpdate={updatePillarFull} t={t}/>;
  }

  return (
    <div style={{minHeight:"100dvh",background:t.bg,paddingBottom:40}}>
      <div style={{background:t.surface,borderBottom:"1px solid "+t.border,padding:"max(env(safe-area-inset-top,44px),44px) 20px 0"}}>
        <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:24}}>
          <span style={{fontFamily:"'Playfair Display',Georgia,serif",fontSize:26,fontWeight:700,color:t.text}}>Mi vida</span>
          <div style={{display:"flex",gap:8}}>
            <button onClick={function(){exportToCSV(data.pillars);}} style={{background:t.surface2,border:"1px solid "+t.border2,borderRadius:10,padding:"7px 12px",cursor:"pointer",color:t.text2,fontSize:12,fontFamily:"'DM Sans',sans-serif",fontWeight:500}}>Exportar CSV</button>
            <button onClick={props.toggleDark} style={{background:t.surface2,border:"1px solid "+t.border2,borderRadius:10,padding:"7px 10px",cursor:"pointer",color:t.text2,fontSize:15}}>{props.darkMode?"Sol":"Luna"}</button>
          </div>
        </div>
        <div style={{display:"flex",flexDirection:"column",alignItems:"center",paddingBottom:32}}>
          <div style={{position:"relative",width:150,height:150,marginBottom:10}}>
            <ScoreRing score={globalScore} size={150} t={t}/>
            <div style={{position:"absolute",inset:0,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center"}}>
              <span style={{fontFamily:"'Playfair Display',Georgia,serif",fontSize:42,fontWeight:800,color:scoreColor,lineHeight:1}}>{globalScore!==null?globalScore:"--"}</span>
              {globalScore!==null && <span style={{fontSize:14,color:scoreColor,fontWeight:600,fontFamily:"'DM Sans',sans-serif"}}>/100</span>}
            </div>
          </div>
          <p style={{fontFamily:"'DM Sans',sans-serif",fontSize:13,color:t.text2,margin:0,fontWeight:500}}>Score general de vida</p>
          {totalWeight > 0 && totalWeight !== 100 && (
            <p style={{fontFamily:"'DM Sans',sans-serif",fontSize:11,color:t.red,margin:"6px 0 0",fontWeight:500}}>Los pesos suman {totalWeight}% - deberian sumar 100%</p>
          )}
        </div>
      </div>

      <div style={{padding:"16px 20px"}}>
        {data.pillars.length===0 && (
          <div style={{textAlign:"center",padding:"60px 0"}}>
            <p style={{fontSize:44,marginBottom:16}}>?#127793;</p>
            <p style={{fontFamily:"'Playfair Display',Georgia,serif",fontSize:20,fontWeight:700,color:t.text2,marginBottom:8}}>Empieza a construir tu vida</p>
            <p style={{fontFamily:"'DM Sans',sans-serif",fontSize:14,color:t.text3,maxWidth:240,margin:"0 auto",lineHeight:1.6}}>Crea tus pilares de vida y empieza a trackear tus objetivos</p>
          </div>
        )}
        {data.pillars.map(function(p){
          var ps = calcPillarScore(p);
          var psColor = ps===null?t.text3:ps>=70?t.green:ps>=40?"#F97316":t.red;
          var activeObjs = (p.objectives||[]).filter(function(o){return o.active!==false;}).length;
          var todayEntries = 0;
          (p.objectives||[]).forEach(function(o){(o.metrics||[]).forEach(function(m){if(m.entries[todayStr()]!==undefined)todayEntries++;});});
          return (
            <div key={p.id} style={{background:t.surface,borderRadius:16,marginBottom:10,border:"1px solid "+t.border,overflow:"hidden",cursor:"pointer",transition:"transform 0.1s, opacity 0.1s"}}
              onClick={function(){setActivePillarId(p.id);}}
              onMouseDown={function(e){e.currentTarget.style.transform="scale(0.985)";}}
              onMouseUp={function(e){e.currentTarget.style.transform="scale(1)";}}
              onTouchStart={function(e){e.currentTarget.style.transform="scale(0.985)";}}
              onTouchEnd={function(e){e.currentTarget.style.transform="scale(1)";}}
            >
              <div style={{height:3,background:p.color}}/>
              <div style={{padding:"14px 16px"}}>
                <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",gap:8}}>
                  <div style={{flex:1,minWidth:0}}>
                    <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:5,flexWrap:"wrap"}}>
                      <span style={{fontFamily:"'Playfair Display',Georgia,serif",fontSize:16,fontWeight:700,color:t.text}}>{p.name}</span>
                      <Badge label={p.weight+"%"} color={p.color}/>
                    </div>
                    <div style={{display:"flex",gap:12,flexWrap:"wrap"}}>
                      <span style={{fontSize:11,color:t.text3,fontFamily:"'DM Sans',sans-serif"}}>{activeObjs} objetivo{activeObjs!==1?"s":""}</span>
                      {todayEntries>0 && <span style={{fontSize:11,color:t.green,fontFamily:"'DM Sans',sans-serif",fontWeight:500}}>{todayEntries} registro{todayEntries!==1?"s":""} hoy</span>}
                    </div>
                  </div>
                  <div style={{display:"flex",alignItems:"center",gap:10}}>
                    <span style={{fontSize:24,fontWeight:800,color:psColor,fontFamily:"'Playfair Display',Georgia,serif"}}>{ps!==null?ps+"%":"--"}</span>
                    <div onClick={function(e){e.stopPropagation();}}>
                      <button onClick={function(e){e.stopPropagation();setEditPillar(p);}} style={{display:"block",background:"none",border:"none",cursor:"pointer",color:t.text3,fontSize:14,padding:"3px 5px"}}>?#9998;</button>
                      <button onClick={function(e){e.stopPropagation();deletePillar(p.id);}} style={{display:"block",background:"none",border:"none",cursor:"pointer",color:t.red+"66",fontSize:13,padding:"3px 5px"}}>x</button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          );
        })}
        <button onClick={function(){setShowAddPillar(true);}} style={{width:"100%",padding:"15px",borderRadius:16,border:"1px dashed "+t.border2,background:"transparent",color:t.text2,cursor:"pointer",fontSize:14,fontFamily:"'DM Sans',sans-serif",fontWeight:500,marginTop:4}}>+ Agregar pilar de vida</button>
      </div>

      {showAddPillar && <Modal title="Nuevo pilar de vida" onClose={function(){setShowAddPillar(false);}} t={t}><PillarForm onSave={addPillar} onClose={function(){setShowAddPillar(false);}} t={t}/></Modal>}
      {editPillar && <Modal title="Editar pilar" onClose={function(){setEditPillar(null);}} t={t}><PillarForm pillar={editPillar} onSave={function(pd){updatePillar(editPillar.id,pd);}} onClose={function(){setEditPillar(null);}} t={t}/></Modal>}
    </div>
  );
}

export default function App() {
  var [data, setData] = useState(null);
  var [darkMode, setDarkMode] = useState(true);

  useEffect(function() {
    var d = load();
    setData(d);
    if (d.darkMode !== undefined) setDarkMode(d.darkMode);
  }, []);

  var updateData = useCallback(function(newData) {
    setData(newData);
    save(newData);
  }, []);

  function toggleDark() {
    setData(function(prev) {
      var nd = !darkMode;
      setDarkMode(nd);
      var updated = Object.assign({},prev,{darkMode:nd});
      save(updated);
      return updated;
    });
  }

  if (!data) {
    return (
      <div style={{minHeight:"100dvh",background:"#0A0A0A",display:"flex",alignItems:"center",justifyContent:"center"}}>
        <div style={{width:36,height:36,borderRadius:"50%",border:"2px solid rgba(255,255,255,0.1)",borderTopColor:"#22C55E",animation:"spin 0.8s linear infinite"}}/>
        <style>{"@keyframes spin{to{transform:rotate(360deg)}}"}</style>
      </div>
    );
  }

  var t = darkMode ? darkTheme : lightTheme;

  return (
    <div style={{background:t.bg,minHeight:"100dvh"}}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;800&family=DM+Sans:wght@400;500;600;700&display=swap');
        @keyframes fadeIn{from{opacity:0}to{opacity:1}}
        @keyframes slideUp{from{transform:translateY(60px);opacity:0}to{transform:translateY(0);opacity:1}}
        @keyframes spin{to{transform:rotate(360deg)}}
        *{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent;}
        input[type=number]::-webkit-inner-spin-button,input[type=number]::-webkit-outer-spin-button{-webkit-appearance:none;}
        input[type=number]{-moz-appearance:textfield;}
        body{background:${t.bg};}
        ::-webkit-scrollbar{width:4px;}
        ::-webkit-scrollbar-track{background:transparent;}
        ::-webkit-scrollbar-thumb{background:${t.border2};border-radius:2px;}
      `}</style>
      <HomeScreen data={data} onUpdateData={updateData} darkMode={darkMode} toggleDark={toggleDark} t={t}/>
    </div>
  );
}
