import { useState, useMemo } from "react";
import { AreaChart, Area, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, Cell } from "recharts";

const FontLink = () => (
  <style>{`
    @import url('https://fonts.googleapis.com/css2?family=Syne:wght@700;800&family=DM+Mono:wght@400;500&family=Outfit:wght@300;400;500;600&display=swap');
    * { box-sizing: border-box; }
    input, select { font-family: 'Outfit', sans-serif; }
    ::-webkit-scrollbar { width: 4px; } ::-webkit-scrollbar-track { background: transparent; } ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.1); border-radius: 2px; }
  `}</style>
);

const CAMPAIGNS = [
  { id:1, name:"Letná kolekcia", audience:"Ženy 25–44", status:"active", budget:15, spent:12.40, reach:4820, impressions:9640, clicks:312, ctr:3.24, cpc:0.040, cpm:1.29, conversions:18, cpa:0.69, roas:4.2, frequency:2.0, color:"#F59E0B",
    trend:[{d:"Po",clicks:42,conv:2,spent:1.8},{d:"Ut",clicks:55,conv:3,spent:2.1},{d:"St",clicks:48,conv:3,spent:1.9},{d:"Št",clicks:61,conv:4,spent:2.2},{d:"Pi",clicks:53,conv:3,spent:2.0},{d:"So",clicks:40,conv:2,spent:1.5},{d:"Ne",clicks:13,conv:1,spent:0.9}]},
  { id:2, name:"Remarketing", audience:"Návštevníci webu", status:"active", budget:8, spent:7.20, reach:1240, impressions:3720, clicks:198, ctr:5.32, cpc:0.036, cpm:1.94, conversions:31, cpa:0.23, roas:8.7, frequency:3.0, color:"#10B981",
    trend:[{d:"Po",clicks:28,conv:4,spent:1.0},{d:"Ut",clicks:30,conv:5,spent:1.1},{d:"St",clicks:27,conv:5,spent:1.0},{d:"Št",clicks:32,conv:6,spent:1.1},{d:"Pi",clicks:29,conv:5,spent:1.0},{d:"So",clicks:35,conv:4,spent:1.3},{d:"Ne",clicks:17,conv:2,spent:0.7}]},
  { id:3, name:"Brand Awareness", audience:"Nová kolekcia", status:"paused", budget:20, spent:18.90, reach:12400, impressions:31200, clicks:520, ctr:1.67, cpc:0.036, cpm:0.61, conversions:9, cpa:2.10, roas:1.8, frequency:2.5, color:"#6366F1",
    trend:[{d:"Po",clicks:74,conv:1,spent:2.7},{d:"Ut",clicks:86,conv:2,spent:3.1},{d:"St",clicks:77,conv:1,spent:2.8},{d:"Št",clicks:88,conv:2,spent:3.2},{d:"Pi",clicks:82,conv:2,spent:3.0},{d:"So",clicks:68,conv:1,spent:2.5},{d:"Ne",clicks:45,conv:0,spent:1.6}]},
  { id:4, name:"Výpredaj", audience:"Muži 30–50", status:"ended", budget:10, spent:10.00, reach:3100, impressions:7440, clicks:241, ctr:3.24, cpc:0.041, cpm:1.34, conversions:14, cpa:0.71, roas:3.5, frequency:2.4, color:"#EC4899",
    trend:[{d:"Po",clicks:34,conv:2,spent:1.4},{d:"Ut",clicks:38,conv:2,spent:1.6},{d:"St",clicks:36,conv:2,spent:1.5},{d:"Št",clicks:44,conv:3,spent:1.8},{d:"Pi",clicks:47,conv:3,spent:1.9},{d:"So",clicks:42,conv:2,spent:1.8}]},
];

function healthScore(c) {
  const r = c.roas >= 6?100:c.roas>=4?85:c.roas>=2.5?65:c.roas>=1.5?40:20;
  const t = c.ctr  >= 4?100:c.ctr >=2.5?80:c.ctr >=1.5?55:c.ctr >=0.8?35:15;
  const p = c.cpa  <=0.3?100:c.cpa<=0.8?80:c.cpa<=2?55:c.cpa<=4?35:15;
  return Math.round(r*0.45+t*0.30+p*0.25);
}
function verdict(sc) {
  if(sc>=80) return {emoji:"🔥",text:"Výborná",desc:"Táto reklama funguje skvele!",color:"#10B981",bg:"rgba(16,185,129,0.12)",border:"rgba(16,185,129,0.25)"};
  if(sc>=60) return {emoji:"⚡",text:"Dobrá",desc:"Solídna výkonnosť",color:"#F59E0B",bg:"rgba(245,158,11,0.12)",border:"rgba(245,158,11,0.25)"};
  if(sc>=40) return {emoji:"⚠️",text:"Priemerná",desc:"Má priestor na zlepšenie",color:"#F97316",bg:"rgba(249,115,22,0.12)",border:"rgba(249,115,22,0.25)"};
  return         {emoji:"❌",text:"Slabá",desc:"Vyžaduje okamžitú pozornosť",color:"#F43F5E",bg:"rgba(244,63,94,0.12)",border:"rgba(244,63,94,0.25)"};
}

function Ring({score,color,size=80}){
  const r=(size-10)/2, circ=2*Math.PI*r, dash=(score/100)*circ;
  return(
    <svg width={size} height={size} style={{transform:"rotate(-90deg)",flexShrink:0}}>
      <circle cx={size/2} cy={size/2} r={r} fill="none" stroke="rgba(255,255,255,0.06)" strokeWidth={7}/>
      <circle cx={size/2} cy={size/2} r={r} fill="none" stroke={color} strokeWidth={7} strokeLinecap="round" strokeDasharray={`${dash} ${circ}`}/>
    </svg>
  );
}

const TT = ({active,payload,label})=>{
  if(!active||!payload?.length) return null;
  return <div style={{background:"#0F172A",border:"1px solid rgba(255,255,255,0.1)",borderRadius:10,padding:"10px 14px",fontFamily:"'Outfit',sans-serif",fontSize:12,color:"#fff"}}>
    <div style={{color:"#64748B",marginBottom:6}}>{label}</div>
    {payload.map(p=><div key={p.name} style={{color:p.color,marginBottom:2}}>{p.name}: <b>{p.value}</b></div>)}
  </div>;
};

const EMPTY={name:"",audience:"",status:"active",budget:"",spent:"",reach:"",impressions:"",clicks:"",ctr:"",cpc:"",cpm:"",conversions:"",cpa:"",roas:"",frequency:"",color:"#6366F1"};
const STATUS_CFG={active:{dot:"#10B981",label:"Beží"},paused:{dot:"#F59E0B",label:"Pausa"},ended:{dot:"#475569",label:"Hotová"}};

export default function App(){
  const[campaigns,setCampaigns]=useState(CAMPAIGNS);
  const[sel,setSel]=useState(CAMPAIGNS[0]);
  const[tab,setTab]=useState("kampane");
  const[filter,setFilter]=useState("all");
  const[modal,setModal]=useState(null);
  const[form,setForm]=useState(EMPTY);
  const[editId,setEditId]=useState(null);

  const filtered=filter==="all"?campaigns:campaigns.filter(c=>c.status===filter);
  const totals=useMemo(()=>({
    spent:campaigns.reduce((s,c)=>s+c.spent,0),
    reach:campaigns.reduce((s,c)=>s+c.reach,0),
    conversions:campaigns.reduce((s,c)=>s+c.conversions,0),
    avgRoas:campaigns.reduce((s,c)=>s+c.roas,0)/campaigns.length,
    avgH:Math.round(campaigns.reduce((s,c)=>s+healthScore(c),0)/campaigns.length),
  }),[campaigns]);

  const sc=sel?healthScore(sel):0;
  const vd=sel?verdict(sc):null;

  function openAdd(){setForm(EMPTY);setModal("add");}
  function openEdit(c){setForm({...c});setEditId(c.id);setModal("edit");}
  function closeModal(){setModal(null);setEditId(null);}
  function del(id){setCampaigns(p=>p.filter(c=>c.id!==id));if(sel?.id===id)setSel(campaigns.find(c=>c.id!==id)??null);}
  function save(){
    const n=k=>parseFloat(form[k])||0;
    const e={...form,budget:n("budget"),spent:n("spent"),reach:n("reach"),impressions:n("impressions"),clicks:n("clicks"),ctr:n("ctr"),cpc:n("cpc"),cpm:n("cpm"),conversions:n("conversions"),cpa:n("cpa"),roas:n("roas"),frequency:n("frequency"),trend:[]};
    if(modal==="add"){e.id=Date.now();setCampaigns(p=>[e,...p]);setSel(e);}
    else{setCampaigns(p=>p.map(c=>c.id===editId?{...e,id:editId,trend:c.trend}:c));if(sel?.id===editId)setSel({...e,id:editId});}
    closeModal();
  }

  const barData=campaigns.map(c=>({name:c.name,ROAS:c.roas,CTR:c.ctr,Zdravie:healthScore(c),color:c.color}));

  return(
    <div style={{minHeight:"100vh",background:"#070B14",fontFamily:"'Outfit',sans-serif",color:"#E2E8F0",position:"relative"}}>
      <FontLink/>
      {/* bg glows */}
      <div style={{position:"fixed",inset:0,pointerEvents:"none",overflow:"hidden",zIndex:0}}>
        <div style={{position:"absolute",width:700,height:700,borderRadius:"50%",background:"radial-gradient(circle,rgba(99,102,241,0.07) 0%,transparent 65%)",top:-200,right:-200}}/>
        <div style={{position:"absolute",width:500,height:500,borderRadius:"50%",background:"radial-gradient(circle,rgba(16,185,129,0.05) 0%,transparent 65%)",bottom:-100,left:-100}}/>
      </div>

      <div style={{position:"relative",zIndex:1,maxWidth:1300,margin:"0 auto",padding:"28px 20px"}}>

        {/* HEADER */}
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:32}}>
          <div>
            <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:5}}>
              <div style={{width:38,height:38,background:"#1877F2",borderRadius:10,display:"flex",alignItems:"center",justifyContent:"center",fontSize:22,fontWeight:900,color:"#fff",fontFamily:"sans-serif",lineHeight:1}}>f</div>
              <span style={{fontFamily:"'Syne',sans-serif",fontSize:24,fontWeight:800,color:"#fff",letterSpacing:"-0.02em"}}>Ads Dashboard</span>
            </div>
            <div style={{fontSize:13,color:"#475569"}}>Výkonnosť tvojich Facebook reklám na jednom mieste</div>
          </div>
          <button onClick={openAdd} style={{background:"linear-gradient(135deg,#1877F2,#4F46E5)",color:"#fff",border:"none",borderRadius:12,padding:"11px 22px",fontSize:13,fontWeight:600,cursor:"pointer",boxShadow:"0 4px 20px rgba(79,70,229,0.4)",display:"flex",alignItems:"center",gap:6}}>
            <span style={{fontSize:17,lineHeight:1}}>＋</span> Nová kampaň
          </button>
        </div>

        {/* KPI STRIP */}
        <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(165px,1fr))",gap:12,marginBottom:26}}>
          {[
            {icon:"💰",label:"Celkové výdavky",val:`€${totals.spent.toFixed(2)}`,sub:"za všetky kampane",accent:"#F59E0B"},
            {icon:"👥",label:"Ľudí oslovených",val:totals.reach.toLocaleString("sk"),sub:"unikátnych osôb",accent:"#6366F1"},
            {icon:"🛒",label:"Nákupy / Leady",val:totals.conversions,sub:"celkový počet",accent:"#10B981"},
            {icon:"📈",label:"Priem. ROAS",val:`${totals.avgRoas.toFixed(1)}×`,sub:"návratnosť výdavkov",accent:"#EC4899"},
            {icon:"💪",label:"Zdravie kampaní",val:`${totals.avgH}%`,sub:"celková výkonnosť",accent:totals.avgH>=70?"#10B981":totals.avgH>=45?"#F59E0B":"#F43F5E"},
          ].map(k=>(
            <div key={k.label} style={{background:"rgba(255,255,255,0.03)",border:"1px solid rgba(255,255,255,0.07)",borderRadius:16,padding:"16px 18px",position:"relative",overflow:"hidden"}}>
              <div style={{position:"absolute",top:0,left:0,right:0,height:2,background:`linear-gradient(90deg,${k.accent},transparent)`}}/>
              <div style={{fontSize:22,marginBottom:8}}>{k.icon}</div>
              <div style={{fontSize:10,color:"#475569",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:3}}>{k.label}</div>
              <div style={{fontSize:26,fontWeight:500,color:k.accent,fontFamily:"'DM Mono',monospace",lineHeight:1}}>{k.val}</div>
              <div style={{fontSize:10,color:"#334155",marginTop:5}}>{k.sub}</div>
            </div>
          ))}
        </div>

        {/* TABS + FILTER */}
        <div style={{display:"flex",gap:8,marginBottom:20,alignItems:"center",flexWrap:"wrap"}}>
          {[["kampane","🗂️ Kampane"],["grafy","📊 Grafy"]].map(([k,l])=>(
            <button key={k} onClick={()=>setTab(k)} style={{background:tab===k?"rgba(99,102,241,0.18)":"transparent",border:`1px solid ${tab===k?"#6366F1":"rgba(255,255,255,0.08)"}`,color:tab===k?"#fff":"#475569",borderRadius:10,padding:"8px 18px",fontSize:13,fontWeight:500,cursor:"pointer"}}>{l}</button>
          ))}
          <div style={{width:1,height:22,background:"rgba(255,255,255,0.08)",margin:"0 4px"}}/>
          {[["all","Všetky"],["active","Aktívne 🟢"],["paused","Pozastavené 🟡"],["ended","Ukončené ⚫"]].map(([k,l])=>(
            <button key={k} onClick={()=>setFilter(k)} style={{background:filter===k?"rgba(255,255,255,0.07)":"transparent",border:`1px solid ${filter===k?"rgba(255,255,255,0.2)":"rgba(255,255,255,0.06)"}`,color:filter===k?"#fff":"#475569",borderRadius:20,padding:"5px 13px",fontSize:11,cursor:"pointer"}}>{l}</button>
          ))}
        </div>

        {/* ═══ KAMPANE TAB ═══ */}
        {tab==="kampane"&&(
          <div style={{display:"grid",gridTemplateColumns:"minmax(290px,400px) 1fr",gap:18,alignItems:"start"}}>

            {/* Campaign list */}
            <div style={{display:"flex",flexDirection:"column",gap:10}}>
              {filtered.map(c=>{
                const csc=healthScore(c), cvd=verdict(csc), isA=sel?.id===c.id, st=STATUS_CFG[c.status];
                return(
                  <div key={c.id} onClick={()=>setSel(c)} style={{background:isA?"rgba(255,255,255,0.055)":"rgba(255,255,255,0.022)",border:`1.5px solid ${isA?c.color+"55":"rgba(255,255,255,0.07)"}`,borderRadius:18,padding:"15px 16px",cursor:"pointer",boxShadow:isA?`0 0 28px ${c.color}14`:"none",transition:"all 0.2s"}}>
                    <div style={{display:"flex",alignItems:"center",gap:12}}>
                      <div style={{position:"relative",flexShrink:0}}>
                        <Ring score={csc} color={c.color} size={62}/>
                        <div style={{position:"absolute",inset:0,display:"flex",alignItems:"center",justifyContent:"center",fontSize:13,fontWeight:600,color:"#fff",fontFamily:"'DM Mono',monospace"}}>{csc}</div>
                      </div>
                      <div style={{flex:1,minWidth:0}}>
                        <div style={{fontWeight:700,fontSize:14,color:"#fff",marginBottom:2,fontFamily:"'Syne',sans-serif",whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis"}}>{c.name}</div>
                        <div style={{fontSize:11,color:"#475569",marginBottom:7}}>{c.audience}</div>
                        <div style={{display:"inline-flex",alignItems:"center",gap:5,background:cvd.bg,border:`1px solid ${cvd.border}`,borderRadius:20,padding:"3px 10px"}}>
                          <span style={{fontSize:12}}>{cvd.emoji}</span>
                          <span style={{fontSize:11,fontWeight:600,color:cvd.color}}>{cvd.text}</span>
                        </div>
                      </div>
                      <div style={{display:"flex",flexDirection:"column",alignItems:"flex-end",gap:8,flexShrink:0}}>
                        <div style={{display:"flex",alignItems:"center",gap:5}}>
                          <div style={{width:6,height:6,borderRadius:"50%",background:st.dot}}/>
                          <span style={{fontSize:10,color:"#475569"}}>{st.label}</span>
                        </div>
                        <div style={{display:"flex",gap:5}} onClick={e=>e.stopPropagation()}>
                          <button onClick={()=>openEdit(c)} style={{background:"rgba(255,255,255,0.07)",border:"none",borderRadius:7,padding:"4px 8px",cursor:"pointer",fontSize:11,color:"#94A3B8"}}>✏️</button>
                          <button onClick={()=>del(c.id)} style={{background:"rgba(244,63,94,0.08)",border:"none",borderRadius:7,padding:"4px 8px",cursor:"pointer",fontSize:11,color:"#F43F5E"}}>🗑</button>
                        </div>
                      </div>
                    </div>
                    {/* mini bars */}
                    <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:8,marginTop:13}}>
                      {[
                        {l:"ROAS",v:c.roas+"×",pct:Math.min(100,c.roas/10*100),col:c.roas>=4?"#10B981":c.roas>=2?"#F59E0B":"#F43F5E"},
                        {l:"CTR",v:c.ctr+"%",pct:Math.min(100,c.ctr/8*100),col:c.ctr>=3?"#10B981":c.ctr>=1.5?"#F59E0B":"#F43F5E"},
                        {l:"Konv.",v:c.conversions,pct:Math.min(100,c.conversions/50*100),col:"#6366F1"},
                      ].map(m=>(
                        <div key={m.l}>
                          <div style={{display:"flex",justifyContent:"space-between",marginBottom:4}}>
                            <span style={{fontSize:9,color:"#475569",textTransform:"uppercase",letterSpacing:"0.08em"}}>{m.l}</span>
                            <span style={{fontSize:11,color:m.col,fontFamily:"'DM Mono',monospace",fontWeight:500}}>{m.v}</span>
                          </div>
                          <div style={{height:3,background:"rgba(255,255,255,0.06)",borderRadius:2}}>
                            <div style={{width:`${m.pct}%`,height:"100%",background:m.col,borderRadius:2}}/>
                          </div>
                        </div>
                      ))}
                    </div>
                  </div>
                );
              })}
            </div>

            {/* Detail panel */}
            {sel&&(
              <div style={{display:"flex",flexDirection:"column",gap:14,position:"sticky",top:20}}>
                {/* hero card */}
                <div style={{background:`linear-gradient(135deg,${sel.color}16,rgba(255,255,255,0.02))`,border:`1px solid ${sel.color}30`,borderRadius:20,padding:"22px 22px 18px"}}>
                  <div style={{display:"flex",alignItems:"flex-start",gap:16,marginBottom:18}}>
                    <div style={{position:"relative",flexShrink:0}}>
                      <Ring score={sc} color={sel.color} size={90}/>
                      <div style={{position:"absolute",inset:0,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center"}}>
                        <div style={{fontSize:20,fontWeight:700,color:"#fff",fontFamily:"'DM Mono',monospace",lineHeight:1}}>{sc}</div>
                        <div style={{fontSize:8,color:"#64748B",textTransform:"uppercase",letterSpacing:"0.1em",marginTop:1}}>skóre</div>
                      </div>
                    </div>
                    <div style={{flex:1}}>
                      <div style={{fontFamily:"'Syne',sans-serif",fontSize:20,fontWeight:800,color:"#fff",marginBottom:3,lineHeight:1.2}}>{sel.name}</div>
                      <div style={{fontSize:12,color:"#64748B",marginBottom:10}}>{sel.audience}</div>
                      <div style={{display:"inline-flex",alignItems:"center",gap:7,background:vd.bg,border:`1px solid ${vd.border}`,borderRadius:20,padding:"6px 14px"}}>
                        <span style={{fontSize:16}}>{vd.emoji}</span>
                        <div>
                          <div style={{fontSize:13,fontWeight:700,color:vd.color,lineHeight:1}}>{vd.text}</div>
                          <div style={{fontSize:10,color:vd.color,opacity:0.7,marginTop:1}}>{vd.desc}</div>
                        </div>
                      </div>
                    </div>
                  </div>
                  {/* budget bar */}
                  <div>
                    <div style={{display:"flex",justifyContent:"space-between",marginBottom:6}}>
                      <span style={{fontSize:11,color:"#475569"}}>Využitý rozpočet dnes</span>
                      <span style={{fontSize:11,color:"#fff",fontFamily:"'DM Mono',monospace"}}>€{sel.spent.toFixed(2)} / €{sel.budget}</span>
                    </div>
                    <div style={{height:6,background:"rgba(255,255,255,0.06)",borderRadius:3}}>
                      <div style={{width:`${Math.min(100,sel.spent/sel.budget*100)}%`,height:"100%",background:`linear-gradient(90deg,${sel.color},${sel.color}99)`,borderRadius:3}}/>
                    </div>
                  </div>
                </div>

                {/* stats grid */}
                <div style={{display:"grid",gridTemplateColumns:"repeat(2,1fr)",gap:9}}>
                  {[
                    {icon:"👁",label:"Dosah",val:sel.reach.toLocaleString("sk"),explain:"ľudí videlo reklamu",accent:"#6366F1"},
                    {icon:"🖱",label:"Kliknutia",val:sel.clicks,explain:"koľkokrát klikli",accent:"#F59E0B"},
                    {icon:"⚡",label:"CTR – miera záujmu",val:`${sel.ctr}%`,explain:sel.ctr>=3?"✓ Výborný záujem":sel.ctr>=1.5?"~ Priemerný záujem":"✗ Nízky záujem",accent:sel.ctr>=3?"#10B981":sel.ctr>=1.5?"#F59E0B":"#F43F5E"},
                    {icon:"🛒",label:"Konverzie",val:sel.conversions,explain:"nákupy alebo leady",accent:"#10B981"},
                    {icon:"📈",label:"ROAS",val:`${sel.roas}×`,explain:sel.roas>=4?`✓ Za €1 späť €${sel.roas}`:sel.roas>=2?"~ Priemerná výnosnosť":"✗ Reklama produkuje stratu",accent:sel.roas>=4?"#10B981":sel.roas>=2?"#F59E0B":"#F43F5E"},
                    {icon:"💸",label:"CPA – cena/konverziu",val:`€${sel.cpa.toFixed(2)}`,explain:sel.cpa<=0.8?"✓ Lacno získaná":sel.cpa<=2?"~ Priemerná cena":"✗ Drahá konverzia",accent:sel.cpa<=0.8?"#10B981":sel.cpa<=2?"#F59E0B":"#F43F5E"},
                    {icon:"📢",label:"CPM – cena/1000 zobr.",val:`€${sel.cpm.toFixed(2)}`,explain:"čím nižšie, tým lepšie",accent:"#94A3B8"},
                    {icon:"🔄",label:"Frekvencia",val:`${sel.frequency}×`,explain:sel.frequency<=2.5?"✓ Zdravá frekvencia":sel.frequency<=4?"~ Trochu veľa":"✗ Ľudia vidia príliš veľa",accent:sel.frequency<=2.5?"#10B981":"#F97316"},
                  ].map(m=>(
                    <div key={m.label} style={{background:"rgba(255,255,255,0.03)",border:"1px solid rgba(255,255,255,0.07)",borderRadius:13,padding:"12px 14px"}}>
                      <div style={{fontSize:11,color:"#475569",marginBottom:3,display:"flex",alignItems:"center",gap:5}}>
                        <span style={{fontSize:13}}>{m.icon}</span> {m.label}
                      </div>
                      <div style={{fontSize:21,fontWeight:500,color:m.accent,fontFamily:"'DM Mono',monospace",lineHeight:1,marginBottom:4}}>{m.val}</div>
                      <div style={{fontSize:10,color:m.accent,opacity:0.75}}>{m.explain}</div>
                    </div>
                  ))}
                </div>

                {/* trend chart */}
                {sel.trend?.length>0&&(
                  <div style={{background:"rgba(255,255,255,0.02)",border:"1px solid rgba(255,255,255,0.06)",borderRadius:16,padding:"16px 14px"}}>
                    <div style={{fontSize:11,color:"#475569",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:14,fontWeight:600}}>📅 Priebeh posledných dní</div>
                    <ResponsiveContainer width="100%" height={150}>
                      <AreaChart data={sel.trend} margin={{top:5,right:5,left:-25,bottom:0}}>
                        <defs>
                          <linearGradient id="g1" x1="0" y1="0" x2="0" y2="1">
                            <stop offset="5%" stopColor={sel.color} stopOpacity={0.3}/><stop offset="95%" stopColor={sel.color} stopOpacity={0}/>
                          </linearGradient>
                          <linearGradient id="g2" x1="0" y1="0" x2="0" y2="1">
                            <stop offset="5%" stopColor="#10B981" stopOpacity={0.3}/><stop offset="95%" stopColor="#10B981" stopOpacity={0}/>
                          </linearGradient>
                        </defs>
                        <CartesianGrid strokeDasharray="3 3" stroke="rgba(255,255,255,0.04)"/>
                        <XAxis dataKey="d" tick={{fill:"#475569",fontSize:11}} axisLine={false} tickLine={false}/>
                        <YAxis tick={{fill:"#475569",fontSize:10}} axisLine={false} tickLine={false}/>
                        <Tooltip content={<TT/>}/>
                        <Area type="monotone" dataKey="clicks" name="Kliknutia" stroke={sel.color} fill="url(#g1)" strokeWidth={2} dot={false}/>
                        <Area type="monotone" dataKey="conv" name="Konverzie" stroke="#10B981" fill="url(#g2)" strokeWidth={2} dot={false}/>
                      </AreaChart>
                    </ResponsiveContainer>
                    <div style={{display:"flex",gap:14,marginTop:8}}>
                      {[[sel.color,"Kliknutia"],["#10B981","Konverzie"]].map(([col,lbl])=>(
                        <div key={lbl} style={{display:"flex",alignItems:"center",gap:5}}>
                          <div style={{width:10,height:3,background:col,borderRadius:2}}/><span style={{fontSize:11,color:"#475569"}}>{lbl}</span>
                        </div>
                      ))}
                    </div>
                  </div>
                )}
              </div>
            )}
          </div>
        )}

        {/* ═══ GRAFY TAB ═══ */}
        {tab==="grafy"&&(
          <div style={{display:"flex",flexDirection:"column",gap:18}}>
            {/* verdicts strip */}
            <div style={{display:"flex",gap:10,flexWrap:"wrap"}}>
              {campaigns.map(c=>{const csc=healthScore(c),cvd=verdict(csc);return(
                <div key={c.id} style={{background:cvd.bg,border:`1px solid ${cvd.border}`,borderRadius:12,padding:"10px 16px",display:"flex",alignItems:"center",gap:10,flex:"1 1 200px"}}>
                  <Ring score={csc} color={cvd.color} size={44}/>
                  <div style={{position:"absolute"}}/>
                  <div>
                    <div style={{fontSize:13,fontWeight:700,color:"#fff",fontFamily:"'Syne',sans-serif"}}>{c.name}</div>
                    <div style={{fontSize:12,color:cvd.color,fontWeight:600}}>{cvd.emoji} {cvd.text} · {csc}/100</div>
                  </div>
                </div>
              );})}
            </div>

            <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(320px,1fr))",gap:16}}>
              {/* Health */}
              <div style={{background:"rgba(255,255,255,0.02)",border:"1px solid rgba(255,255,255,0.07)",borderRadius:18,padding:"20px 18px"}}>
                <div style={{fontSize:11,color:"#475569",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:4,fontWeight:600}}>💪 Zdravie kampane (0–100)</div>
                <div style={{fontSize:12,color:"#334155",marginBottom:16}}>Celkové hodnotenie výkonnosti</div>
                <div style={{display:"flex",flexDirection:"column",gap:14}}>
                  {campaigns.map(c=>{const csc=healthScore(c),cvd=verdict(csc);return(
                    <div key={c.id}>
                      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:6}}>
                        <div><span style={{fontSize:13,color:"#fff",fontWeight:500}}>{c.name}</span></div>
                        <div style={{display:"flex",alignItems:"center",gap:8}}>
                          <span style={{fontSize:12,color:cvd.color}}>{cvd.emoji} {cvd.text}</span>
                          <span style={{fontFamily:"'DM Mono',monospace",fontSize:15,fontWeight:600,color:cvd.color}}>{csc}</span>
                        </div>
                      </div>
                      <div style={{height:8,background:"rgba(255,255,255,0.05)",borderRadius:4}}>
                        <div style={{width:`${csc}%`,height:"100%",background:`linear-gradient(90deg,${cvd.color},${cvd.color}77)`,borderRadius:4}}/>
                      </div>
                    </div>
                  );})}
                </div>
              </div>

              {/* ROAS bars */}
              <div style={{background:"rgba(255,255,255,0.02)",border:"1px solid rgba(255,255,255,0.07)",borderRadius:18,padding:"20px 18px"}}>
                <div style={{fontSize:11,color:"#475569",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:4,fontWeight:600}}>📈 ROAS – návratnosť výdavkov</div>
                <div style={{fontSize:12,color:"#334155",marginBottom:16}}>Za každé €1 vložené do reklamy dostaneš späť...</div>
                <ResponsiveContainer width="100%" height={190}>
                  <BarChart data={barData} margin={{top:5,right:10,left:-20,bottom:5}}>
                    <CartesianGrid strokeDasharray="3 3" stroke="rgba(255,255,255,0.04)" vertical={false}/>
                    <XAxis dataKey="name" tick={{fill:"#475569",fontSize:11}} axisLine={false} tickLine={false}/>
                    <YAxis tick={{fill:"#475569",fontSize:10}} axisLine={false} tickLine={false}/>
                    <Tooltip content={<TT/>}/>
                    <Bar dataKey="ROAS" name="ROAS" radius={[8,8,0,0]}>
                      {barData.map((e,i)=><Cell key={i} fill={e.ROAS>=4?"#10B981":e.ROAS>=2?"#F59E0B":"#F43F5E"}/>)}
                    </Bar>
                  </BarChart>
                </ResponsiveContainer>
                <div style={{display:"flex",gap:12,marginTop:4,flexWrap:"wrap"}}>
                  {[["🔥 Výborné ≥4×","#10B981"],["⚡ Dobré 2–4×","#F59E0B"],["❌ Slabé <2×","#F43F5E"]].map(([t,c])=>(
                    <div key={t} style={{display:"flex",alignItems:"center",gap:5}}><div style={{width:8,height:8,borderRadius:2,background:c}}/><span style={{fontSize:11,color:"#475569"}}>{t}</span></div>
                  ))}
                </div>
              </div>

              {/* CTR */}
              <div style={{background:"rgba(255,255,255,0.02)",border:"1px solid rgba(255,255,255,0.07)",borderRadius:18,padding:"20px 18px"}}>
                <div style={{fontSize:11,color:"#475569",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:4,fontWeight:600}}>⚡ CTR – záujem ľudí o reklamu</div>
                <div style={{fontSize:12,color:"#334155",marginBottom:16}}>Koľko % ľudí, čo videlo reklamu, aj kliklo</div>
                <ResponsiveContainer width="100%" height={190}>
                  <BarChart data={barData} margin={{top:5,right:10,left:-20,bottom:5}}>
                    <CartesianGrid strokeDasharray="3 3" stroke="rgba(255,255,255,0.04)" vertical={false}/>
                    <XAxis dataKey="name" tick={{fill:"#475569",fontSize:11}} axisLine={false} tickLine={false}/>
                    <YAxis tick={{fill:"#475569",fontSize:10}} axisLine={false} tickLine={false}/>
                    <Tooltip content={<TT/>}/>
                    <Bar dataKey="CTR" name="CTR %" radius={[8,8,0,0]}>
                      {barData.map((e,i)=><Cell key={i} fill={e.CTR>=3?"#10B981":e.CTR>=1.5?"#F59E0B":"#F43F5E"}/>)}
                    </Bar>
                  </BarChart>
                </ResponsiveContainer>
                <div style={{display:"flex",gap:12,marginTop:4,flexWrap:"wrap"}}>
                  {[["🔥 Výborné ≥3%","#10B981"],["⚡ Dobré 1.5–3%","#F59E0B"],["❌ Slabé <1.5%","#F43F5E"]].map(([t,c])=>(
                    <div key={t} style={{display:"flex",alignItems:"center",gap:5}}><div style={{width:8,height:8,borderRadius:2,background:c}}/><span style={{fontSize:11,color:"#475569"}}>{t}</span></div>
                  ))}
                </div>
              </div>

              {/* Spend vs Conv */}
              <div style={{background:"rgba(255,255,255,0.02)",border:"1px solid rgba(255,255,255,0.07)",borderRadius:18,padding:"20px 18px"}}>
                <div style={{fontSize:11,color:"#475569",textTransform:"uppercase",letterSpacing:"0.1em",marginBottom:4,fontWeight:600}}>🛒 Výdavky vs. Výsledky</div>
                <div style={{fontSize:12,color:"#334155",marginBottom:16}}>Koľko stojí kampaň a koľko prináša</div>
                <div style={{display:"flex",flexDirection:"column",gap:14}}>
                  {campaigns.map(c=>{
                    const mx=Math.max(...campaigns.map(x=>x.spent));
                    const mc=Math.max(...campaigns.map(x=>x.conversions));
                    return(
                      <div key={c.id}>
                        <div style={{display:"flex",justifyContent:"space-between",marginBottom:5}}>
                          <span style={{fontSize:12,color:"#94A3B8",fontWeight:500}}>{c.name}</span>
                          <span style={{fontSize:11,color:"#475569",fontFamily:"'DM Mono',monospace"}}>€{c.spent.toFixed(2)} · {c.conversions} konv.</span>
                        </div>
                        <div style={{display:"flex",gap:3}}>
                          <div style={{height:7,background:"#1877F2",borderRadius:3,width:`${(c.spent/mx)*60}%`,minWidth:4}}/>
                          <div style={{height:7,background:"#10B981",borderRadius:3,width:`${(c.conversions/mc)*40}%`,minWidth:4}}/>
                        </div>
                      </div>
                    );
                  })}
                </div>
                <div style={{display:"flex",gap:12,marginTop:14}}>
                  {[["#1877F2","Výdavky (€)"],["#10B981","Konverzie"]].map(([c,l])=>(
                    <div key={l} style={{display:"flex",alignItems:"center",gap:5}}><div style={{width:10,height:4,background:c,borderRadius:2}}/><span style={{fontSize:11,color:"#475569"}}>{l}</span></div>
                  ))}
                </div>
              </div>
            </div>
          </div>
        )}

        {/* LEGEND / HELP */}
        <div style={{marginTop:24,padding:"13px 16px",background:"rgba(255,255,255,0.02)",border:"1px solid rgba(255,255,255,0.05)",borderRadius:12,display:"flex",flexWrap:"wrap",gap:8,alignItems:"center"}}>
          <span style={{fontSize:11,color:"#334155",marginRight:4}}>💡</span>
          {[
            ["ROAS","koľko € zarobíš za každé €1 v reklame (čím vyššie, tým lepšie)"],
            ["CTR","% ľudí čo klikli (ideálne 2–5%)"],
            ["CPA","cena za 1 konverziu/nákup (čím nižšie, tým lepšie)"],
            ["Frekvencia","koľkokrát videl 1 človek reklamu (ideálne 1–3×)"],
          ].map(([k,v])=>(
            <span key={k} style={{fontSize:11,color:"#475569",background:"rgba(255,255,255,0.035)",borderRadius:6,padding:"3px 9px"}}>
              <b style={{color:"#94A3B8"}}>{k}</b> = {v}
            </span>
          ))}
        </div>
      </div>

      {/* ═══ MODAL ═══ */}
      {modal&&(
        <div style={{position:"fixed",inset:0,background:"rgba(0,0,0,0.78)",backdropFilter:"blur(6px)",display:"flex",alignItems:"center",justifyContent:"center",zIndex:999}} onClick={closeModal}>
          <div style={{background:"#0D1421",border:"1px solid rgba(255,255,255,0.1)",borderRadius:22,padding:"26px 24px",width:"min(680px,95vw)",maxHeight:"88vh",overflowY:"auto"}} onClick={e=>e.stopPropagation()}>
            <div style={{fontFamily:"'Syne',sans-serif",fontSize:18,fontWeight:800,color:"#fff",marginBottom:20}}>{modal==="add"?"➕ Nová kampaň":"✏️ Upraviť kampaň"}</div>
            <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(150px,1fr))",gap:11}}>
              {[
                ["name","Názov kampane","text",true],["audience","Cieľová skupina","text",true],
                ["budget","Denný rozpočet €","number",false],["spent","Výdavky €","number",false],
                ["reach","Dosah (ľudia)","number",false],["impressions","Zobrazenia","number",false],
                ["clicks","Kliknutia","number",false],["ctr","CTR %","number",false],
                ["cpc","CPC €","number",false],["cpm","CPM €","number",false],
                ["conversions","Konverzie","number",false],["cpa","CPA €","number",false],
                ["roas","ROAS","number",false],["frequency","Frekvencia","number",false],
              ].map(([key,label,type,full])=>(
                <div key={key} style={{gridColumn:full?"1 / -1":undefined,display:"flex",flexDirection:"column",gap:4}}>
                  <label style={{fontSize:10,color:"#475569",textTransform:"uppercase",letterSpacing:"0.08em"}}>{label}</label>
                  <input type={type} value={form[key]??""} step={type==="number"?"any":undefined}
                    onChange={e=>setForm(f=>({...f,[key]:e.target.value}))}
                    style={{background:"rgba(255,255,255,0.05)",border:"1px solid rgba(255,255,255,0.1)",borderRadius:9,padding:"9px 12px",color:"#fff",fontSize:13,outline:"none",width:"100%"}}/>
                </div>
              ))}
              <div style={{display:"flex",flexDirection:"column",gap:4}}>
                <label style={{fontSize:10,color:"#475569",textTransform:"uppercase",letterSpacing:"0.08em"}}>Stav</label>
                <select value={form.status} onChange={e=>setForm(f=>({...f,status:e.target.value}))}
                  style={{background:"#0D1421",border:"1px solid rgba(255,255,255,0.1)",borderRadius:9,padding:"9px 12px",color:"#fff",fontSize:13,outline:"none"}}>
                  <option value="active">Aktívna 🟢</option><option value="paused">Pozastavená 🟡</option><option value="ended">Ukončená ⚫</option>
                </select>
              </div>
            </div>
            <div style={{display:"flex",gap:10,marginTop:20,justifyContent:"flex-end"}}>
              <button onClick={closeModal} style={{background:"transparent",border:"1px solid rgba(255,255,255,0.1)",color:"#475569",borderRadius:10,padding:"10px 20px",cursor:"pointer",fontSize:13}}>Zrušiť</button>
              <button onClick={save} style={{background:"linear-gradient(135deg,#1877F2,#4F46E5)",color:"#fff",border:"none",borderRadius:10,padding:"10px 24px",fontWeight:600,fontSize:13,cursor:"pointer"}}>{modal==="add"?"Pridať kampaň":"Uložiť zmeny"}</button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
