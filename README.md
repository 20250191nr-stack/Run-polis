<!DOCTYPE html>
<html lang="ja"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>シティ・エスケープ 3D</title>
<style>html,body{margin:0;height:100%;background:#bfe3ff;overflow:hidden}canvas{position:fixed;left:0;top:0;width:100%;height:100%}#hud{pointer-events:none}
#ov{position:fixed;inset:0;background:rgba(8,12,20,.74);display:flex;align-items:center;justify-content:center;z-index:10;font-family:sans-serif;color:#fff}
#ov .box{width:min(760px,94vw);max-height:88vh;display:flex;flex-direction:column;background:#161e2b;border:2px solid #3a4a63;border-radius:12px;overflow:hidden}
#ov .hd{display:flex;justify-content:space-between;padding:12px 16px;background:#1f2b3d;font-size:18px}
#ov .tabs{display:flex;gap:6px;padding:8px 12px;background:#1a2433;flex-wrap:wrap}
#ov button{background:#2c3b52;color:#fff;border:1px solid #4a5d7c;border-radius:6px;padding:8px 14px;font-size:15px;cursor:pointer;margin:2px}
#ov button.on,#ov button.pri{background:#2f7fd1;border-color:#6aaaf0}
#ov button:disabled{opacity:.4;cursor:not-allowed}
#ov .body{padding:12px;overflow:auto;flex:1}
#ov .ft{padding:10px 12px;background:#1a2433;text-align:right}
#ov .card{background:#1e2a3c;border:1px solid #33455f;border-radius:8px;padding:10px;margin-bottom:10px}
#ov .st{display:flex;align-items:center;gap:8px;font-size:13px;margin:3px 0}
#ov .st span{width:48px}#ov .bar{flex:1;height:8px;background:#0f1520;border-radius:4px}#ov .bar i{display:block;height:100%;background:#4fb3ff;border-radius:4px}
#ov .up{color:#6be08a;font-style:normal}#ov .dn{color:#ff7a7a;font-style:normal}
#ov .sw{display:inline-block;width:30px;height:30px;border-radius:6px;margin:3px;border:2px solid #fff3;cursor:pointer}
#ov .row{display:flex;align-items:center;justify-content:space-between;gap:8px;margin:6px 0;font-size:15px}</style>
</head><body><canvas id="hud"></canvas>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const CELL=400,N=32,SIZE=CELL*N,XMAX=SIZE+2100,RW=100,PI=Math.PI;
const hud=document.getElementById('hud'),g=hud.getContext('2d');
let W,H;
const scene=new THREE.Scene();scene.background=new THREE.Color(0xbfe3ff);scene.fog=new THREE.Fog(0xbfe3ff,500,2100);
const cam=new THREE.PerspectiveCamera(62,1,5,4500);
const ren=new THREE.WebGLRenderer({antialias:true});ren.setPixelRatio(Math.min(devicePixelRatio,2));
document.body.insertBefore(ren.domElement,hud);
function rs(){W=hud.width=innerWidth;H=hud.height=innerHeight;ren.setSize(W,H);cam.aspect=W/H;cam.updateProjectionMatrix()}rs();addEventListener('resize',rs);
const HL=new THREE.HemisphereLight(0xffffff,0xb8d4a8,.95);scene.add(HL);
const sun=new THREE.DirectionalLight(0xfff0d0,.7);sun.position.set(300,600,200);scene.add(sun);
const K={};let state='title',best=0;
addEventListener('keydown',e=>{K[e.code]=1;if(state!='play'){if(e.code=='Digit1')diff=0;if(e.code=='Digit2')diff=1;if(e.code=='Digit3')diff=2}if(e.code=='Space')e.preventDefault();if((e.code=='Enter'||e.code=='KeyR')&&state!='play')start()});
addEventListener('keyup',e=>K[e.code]=0);
const R=(a,b)=>a+Math.random()*(b-a),clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
function hash(i,j){let h=(i*374761393+j*668265263)>>>0;h=((h^(h>>>13))*1274126177)>>>0;return h/4294967296}
const RR=[5,17],RC=[9,19],BV=[4,12,21],BH=[3,12,20],ARCH=17;
const isWater=(i,j)=>RR.includes(j)||RC.includes(i);
const bandH=x=>RC.some(i=>x>i*CELL+RW&&x<(i+1)*CELL),bandV=z=>RR.some(j=>z>j*CELL+RW&&z<(j+1)*CELL);
const bandFz=z=>z>FWZ0&&z<FWZ0+300,hClosed=(x,a)=>!BH.includes(a)&&bandH(x),vClosed=(z,a)=>(!BV.includes(a)&&bandV(z))||(!OVP.includes(a)&&bandFz(z));
const DYNSEG=new Set();
const segClosed0=(ix,iy,nx,ny)=>nx!=ix?hClosed((ix+nx)/2,Math.round((iy-50)/CELL)):vClosed((iy+ny)/2,Math.round((ix-50)/CELL));
const segClosed=(a,b,c,d)=>segClosed0(a,b,c,d)||DYNSEG.has(segKey(a,b,c,d));
const CL=[],BR=[];
for(let a=0;a<N;a++){for(const j of RR)if(!BV.includes(a))CL.push([a*CELL-12,j*CELL+RW,a*CELL+RW+12,(j+1)*CELL]);for(const i of RC)if(!BH.includes(a))CL.push([i*CELL+RW,a*CELL-12,(i+1)*CELL,a*CELL+RW+12])}
for(const a of BH)for(const i of RC)BR.push({h:1,u0:i*CELL+RW,v0:a*CELL-12,v1:a*CELL+RW+12});
for(const a of BV)for(const j of RR)BR.push({h:0,u0:j*CELL+RW,v0:a*CELL-12,v1:a*CELL+RW+12});
const inCL=(x,y)=>CL.some(q=>x>q[0]&&x<q[2]&&y>q[1]&&y<q[3]);
const MR0=700,MB=135/(2*Math.PI),MPH=8*Math.PI,MP1=6*Math.PI,MHP=150/(2*Math.PI),MX=SIZE+1300,MZC=Math.floor(N/2)*CELL+50,MZ=MZC+MR0,MT0=-Math.PI/2,MHW=42;
const mhf=ph=>MHP*Math.min(ph,MP1);
function mInfo(x,z){const dx=x-MX,dz=z-MZ,r=Math.hypot(dx,dz);if(r>MR0+60)return null;if(x<MX&&Math.abs(z-MZC)<60)return null;
 if(r<200)return{free:true,h:mhf(MPH),r,th:0,e:0};
 const th=Math.atan2(dz,dx),p0=((th-MT0)%(2*PI)+2*PI)%(2*PI);let k=Math.round((MR0-MB*p0-r)/(2*PI*MB));k=Math.max(0,Math.min(3,k));const ph=p0+2*PI*k;if(ph>MPH)return null;
 return{r,th,ph,e:r-(MR0-MB*ph),h:mhf(ph)}}
function mRails(c,r){const m=mInfo(c.x,c.y);if(!m||m.free)return 0;const L=MHW-r-1,ae=Math.abs(m.e);if(ae<=L||ae>L+25)return 0;
 const sg=m.e>0?1:-1,ux=Math.cos(m.th)*sg,uy=Math.sin(m.th)*sg,pen=ae-L;c.x-=ux*pen;c.y-=uy*pen;const vn=c.vx*ux+c.vy*uy;if(vn>0){c.vx-=1.5*vn*ux;c.vy-=1.5*vn*uy;return vn}return 0}
function gh(x,z){const m=mInfo(x,z);if(m)return(m.free||Math.abs(m.e)<=49)?m.h:0;const ov=ovH(x,z);if(ov>0)return ov;for(const b of BR){const u=b.h?x:z,v=b.h?z:x;if(u>b.u0&&u<b.u0+300&&v>b.v0&&v<b.v1){const t=(u-b.u0)/300;return ARCH*4*t*(1-t)}}return 0}
const isPark=(i,j)=>blockKind(i,j)=='park';
const PAL=['#f4a6b7','#a6d8f4','#f7e1a0','#b8e6b0','#d4b8f0','#f9c49a','#9fe0d6'];
const CARC=['#f2f2f2','#1f2328','#9ea4ab','#b3262b','#1f4e8c','#d9d9d9','#e4b43c'];
/* ---------- 3D helpers ---------- */
const gBox=new THREE.BoxGeometry(1,1,1),gCyl=new THREE.CylinderGeometry(1,1,1,10),gSph=new THREE.SphereGeometry(1,10,8);
const mc={};const mat=c=>mc[c]||(mc[c]=new THREE.MeshLambertMaterial({color:c}));
const LR=new THREE.MeshBasicMaterial({color:0xff2a3d}),LB=new THREE.MeshBasicMaterial({color:0x2a6bff});
function bx(par,m,x,y,z,w,h,d){const o=new THREE.Mesh(gBox,m);o.position.set(x,y,z);o.scale.set(w,h,d);par.add(o);return o}
function carMesh(type,col){const G=new THREE.Group(),m=mat(col);
 const body=bx(G,m,0,7,0,34,8,18),roof=bx(G,m,-2,17.5,0,13,2,15.5);
 bx(G,mat('#2b3a4a'),-2,14,0,17,5,15.8);
 for(const sx of[-10,11])for(const sz of[-9,9]){const w=new THREE.Mesh(gCyl,mat('#222'));w.rotation.x=PI/2;w.scale.set(3.6,3,3.6);w.position.set(sx,3.5,sz);G.add(w)}
 for(const z of[-6,6]){bx(G,mat('#ffe9a0'),17,8,z,1,2,3);bx(G,mat('#ff4a4a'),-17,8,z,1,2,3)}
 if(type=='p')bx(G,mat('#fff'),0,11.2,0,34.4,.6,4);
 if(type=='police'){bx(G,mat('#222'),0,7,0,10,8.3,18.3);G.l1=bx(G,LR,-2,19.5,-4,4,2,6);G.l2=bx(G,LB,-2,19.5,4,4,2,6)}
 G.tint=[body,roof];return G}
function pedMesh(col){const G=new THREE.Group();const b=new THREE.Mesh(gCyl,mat(col));b.scale.set(3,9,3);b.position.y=5;G.add(b);
 const h=new THREE.Mesh(gSph,mat('#ffe0c0'));h.scale.setScalar(3);h.position.y=12;G.add(h);return G}
const SK=['#ffe0c0','#f1c8a0','#d9a577','#a9744f','#7a4b32'],HR=['#2a1d15','#4a3020','#8a5a2b','#c9a24a','#999999','#1a1a1a'],PANT=['#2e3a59','#444444','#6b5b4a','#1f2a44','#7a8a6a'];
const doors=[],gCoin=new THREE.CylinderGeometry(4.5,4.5,1.2,14),gBar=new THREE.BoxGeometry(10,4,5),GM=new THREE.MeshLambertMaterial({color:0xffc928,emissive:0x7a5200});gCoin.rotateX(PI/2);
function addCoin(x,y,val,keep){const bar=val>=500,mesh=new THREE.Mesh(bar?gBar:gCoin,GM);mesh.scale.setScalar(bar?1.4:1.25);mesh.position.set(x,7,y);scene.add(mesh);coins.push({x,y,val,mesh,ph:R(0,6),keep:!!keep})}
function spawnCoin(a0=600,a1=1300){const s=spawnPt(a0,a1),o=R(-38,38);if(s.h)s.y+=o;else s.x+=o;addCoin(s.x,s.y,Math.random()<.08?500:Math.random()<.3?100:50)}
function policeTarget(c){let best=null,bd=1e9;
 const mp=mInfo(p.x,p.y);
 if(mp&&(mp.free||mp.ph>.3)){const mc=mInfo(c.x,c.y);
  if(mc&&!mc.free){const dir=mp.free||mc.ph<mp.ph?1:-1,ph=clamp(mc.ph+dir*.45,0,MPH),th=MT0+ph,r=MR0-MB*ph;return{x:MX+r*Math.cos(th),y:MZ+r*Math.sin(th)}}
  if(!mc)return{x:MX+40,y:MZC}}
 const test=(row,r)=>{const pos=row?c.y:c.x,pp=row?p.y:p.x,lo=r*CELL+RW,hi=(r+1)*CELL,sd=v=>v<lo?0:v>hi?2:1,a=sd(pos),b=sd(pp);
  if(a==b)return;const dist=Math.abs(pos-(lo+hi)/2);if(dist<bd){bd=dist;best={row,r,a,b}}};
 for(const r of RR)test(true,r);for(const r of RC)test(false,r);
 if(!best)return p;
 const {row,r,a,b}=best,arr=row?BV:BH,pos=row?c.x:c.y;let bi=arr[0],bdist=1e9;for(const i of arr){const d=Math.abs(i*CELL+50-pos);if(d<bdist){bdist=d;bi=i}}
 const end=s=>s==0?r*CELL+50:(r+1)*CELL+50,bc=bi*CELL+50;let e;
 if(a==1)e=end(b);else{e=end(a);if(Math.hypot(row?bc-c.x:e-c.x,row?e-c.y:bc-c.y)<130)e=end(2-a)}
 return row?{x:bc,y:e}:{x:e,y:bc}}
function pedMesh3(col){const G=new THREE.Group(),pk=a=>a[Math.floor(Math.random()*a.length)],sk=mat(pk(SK)),pm=mat(pk(PANT)),sh=mat(pk(['#222222','#eeeeee','#553322']));
 const limb=(m1,m2,z,len,w,y)=>{const pv=new THREE.Group();pv.position.set(0,y,z);bx(pv,m1,0,-len/4,0,w,len/2,w);const kn=new THREE.Group();kn.position.y=-len/2;bx(kn,m2,0,-len/4,0,w*.92,len/2,w*.92);pv.add(kn);pv.k=kn;G.add(pv);return pv};
 G.lL=limb(pm,pm,-1.3,7,2.4,7.2);G.lR=limb(pm,pm,1.3,7,2.4,7.2);
 for(const L of[G.lL,G.lR])bx(L.k,sh,.7,-3.4,0,3.6,1.3,2.6);
 const sl=Math.random()<.5?mat(col):sk;G.aL=limb(sl,sk,-3.4,5.6,1.7,12.6);G.aR=limb(sl,sk,3.4,5.6,1.7,12.6);
 bx(G,mat(col),0,10,0,3.2,6,5.2);
 if(Math.random()<.35)bx(G,mat(pk(['#3a5a8a','#8a3a3a','#3a3a3a'])),-2.4,10.5,0,1.8,5,4);
 const hd=new THREE.Mesh(gSph,sk);hd.scale.set(2.2,2.5,2.2);hd.position.y=15.4;G.add(hd);
 const hr=new THREE.Mesh(gSph,mat(pk(HR)));hr.scale.set(2.35,1.7,2.35);hr.position.set(-.2,16.4,0);G.add(hr);
 G.scale.setScalar(.78);return G}
function pedAI(q,k,dx,dy,d,sp){const fl=q.mode!=3&&d<140&&sp>1;
 if(q.mode!=3&&d<140&&sp>1){q.mode=0;q.door=null;q.tvx=dx/d*2.2;q.tvy=dy/d*2.2;q.tm=30}
 else if(q.mode==3){q.ins-=k;q.vx=q.vy=0;if(q.ins<=0){q.mode=4;q.mesh.visible=true;q.x=q.door.x;q.y=q.door.y;q.tx=q.door.gx;q.ty=q.door.gy}return}
 else if(q.mode>0){const tg=q.mode==2?q.door:{x:q.tx,y:q.ty},ex=tg.x-q.x,ey=tg.y-q.y,ed=Math.hypot(ex,ey);
  if(ed<3){if(q.mode==1)q.mode=2;else if(q.mode==2){q.mode=3;q.ins=R(300,900);q.mesh.visible=false;q.vx=q.vy=0;return}else{q.mode=0;q.tm=0}}
  else{q.tvx=ex/ed*q.ws;q.tvy=ey/ed*q.ws}}
 else{q.tm-=k;if(q.tm<=0){
  if(Math.random()<.35){let best=null,bd=240;for(const D of doors){const dd=Math.hypot(D.gx-q.x,D.gy-q.y);if(dd<bd){bd=dd;best=D}}
   if(best){q.door=best;q.tx=best.gx;q.ty=best.gy;q.mode=1}}
  if(q.mode==0){if(Math.random()<.25){q.tvx=q.tvy=0}else{const a=Math.floor(R(0,4))*PI/2;q.tvx=Math.cos(a)*q.ws;q.tvy=Math.sin(a)*q.ws}q.tm=R(60,200)}}}
 q.vx+=(q.tvx-q.vx)*.12*k;q.vy+=(q.tvy-q.vy)*.12*k;
 const nx=q.x+q.vx*k,ny=q.y+q.vy*k;
 if(q.mode==0&&!fl&&!pedCrossOK(q,nx,ny)){q.vx=q.vy=0;q.tvx=q.tvy=0;q.tm=Math.max(q.tm,20);return}
 if(q.mode>0||onRoad(nx,ny)){q.x=nx;q.y=ny}else{q.tvx=-q.tvx;q.tvy=-q.tvy;q.vx=-q.vx;q.vy=-q.vy;q.tm=0}}
function animPed(q,k){const m=q.mesh;if(q.mode==3)return;const gy=gh(q.x,q.y);
 if(q.hit>0){const y=Math.sin((1-q.hit/45)*PI)*30;m.position.set(q.x,y+.6+gy,q.y);m.rotation.x+=.3*k;m.aL.rotation.z=2.4;m.aR.rotation.z=-2.4;m.lL.rotation.z=.6;m.lR.rotation.z=-.6;m.lL.k.rotation.z=m.lR.k.rotation.z=0;return}
 const sp=Math.hypot(q.vx,q.vy);m.position.set(q.x,.6+gy,q.y);
 if(sp>.08){let d=Math.atan2(q.vy,q.vx)-q.fa;d=((d+PI*3)%(PI*2))-PI;q.fa+=d*.2*Math.min(k,1.5);q.ph+=sp*.42*k;
  const run=clamp((sp-1)/1.4,0,1),A=.45+run*.55,s=Math.sin(q.ph),kn=.5+run*.8;
  m.lL.rotation.z=s*A;m.lR.rotation.z=-s*A;m.lL.k.rotation.z=-Math.max(0,Math.cos(q.ph))*kn;m.lR.k.rotation.z=-Math.max(0,-Math.cos(q.ph))*kn;
  m.aL.rotation.z=-s*A*.9;m.aR.rotation.z=s*A*.9;m.aL.k.rotation.z=m.aR.k.rotation.z=.25+run*.9;
  m.rotation.z=-run*.25;m.position.y+=Math.abs(s)*(.3+run*.5)}
 else{m.lL.rotation.z=m.lR.rotation.z=m.aL.rotation.z=0;m.lL.k.rotation.z=m.lR.k.rotation.z=0;m.aL.k.rotation.z=m.aR.k.rotation.z=.15;m.rotation.z=0;q.idle=(q.idle||0)+k*.03;m.aR.rotation.z=Math.sin(q.idle)*.05}
 m.rotation.y=-q.fa}
const SKN=['#f4d3b5','#e6b98f','#c68e62','#9c6a45','#6e4630','#4a2f20'],HAIRC=['#1a1410','#3a2a1c','#6b4a2a','#b88a4a','#d8c07a','#8a8a8a','#5a2a1a'],TOPC=['#d94a4a','#2f6fb5','#f2f2f2','#222222','#e0a030','#3a9a6a','#8a4fb0','#d86a9a','#6a7a8a','#c9c24a'],BOTC=['#1f3a66','#2b2b30','#3d5a80','#6b5a44','#8a8f99','#a89060','#1a1a1a'];
function pedMesh4(){const G=new THREE.Group(),pk=a=>a[Math.floor(Math.random()*a.length)],sk=mat(pk(SKN)),top=mat(pk(TOPC)),bot=mat(pk(BOTC)),shoe=mat(pk(['#f2f2f2','#222222','#5a3a2a','#cc3333'])),hair=mat(pk(HAIRC)),fem=Math.random()<.45,shorts=Math.random()<.3,sleeveless=Math.random()<.3,longs=!sleeveless&&Math.random()<.4,bd=.9+Math.random()*.25;
 const cy=(par,m,x,y,z,r,h)=>{const o=new THREE.Mesh(gCyl,m);o.position.set(x,y,z);o.scale.set(r,h,r);par.add(o);return o};
 const leg=z=>{const pv=new THREE.Group();pv.position.set(0,.95,z);cy(pv,bot,0,-.23,0,.085,.46);const kn=new THREE.Group();kn.position.y=-.46;cy(kn,shorts?sk:bot,0,-.22,0,.065,.44);bx(kn,shoe,.06,-.455,0,.27,.07,.1);pv.add(kn);pv.k=kn;G.add(pv);return pv};
 const arm=z=>{const pv=new THREE.Group();pv.position.set(0,1.42,z);cy(pv,sleeveless?sk:top,0,-.15,0,.048,.3);const kn=new THREE.Group();kn.position.y=-.3;cy(kn,longs?top:sk,0,-.135,0,.04,.27);bx(kn,sk,0,-.29,0,.06,.09,.05);pv.add(kn);pv.k=kn;G.add(pv);return pv};
 G.lL=leg(-.09*bd);G.lR=leg(.09*bd);G.aL=arm(-.21*bd);G.aR=arm(.21*bd);
 bx(G,bot,0,.93,0,.2,.2,.38*bd);bx(G,top,0,1.2,0,.2,.46,.4*bd);bx(G,top,0,1.38,0,.19,.12,.45*bd);cy(G,sk,0,1.5,0,.045,.1);
 const hd=new THREE.Mesh(gSph,sk);hd.scale.set(.095,.115,.1);hd.position.y=1.63;G.add(hd);
 const hs=fem&&Math.random()<.7?1:Math.floor(Math.random()*5);
 if(hs==0||hs==4||hs==1){const h=new THREE.Mesh(gSph,hair);h.scale.set(hs==4?.13:.1,hs==4?.1:.075,hs==4?.13:.105);h.position.set(-.01,hs==4?1.7:1.69,0);G.add(h);if(hs==1)bx(G,hair,-.07,1.56,0,.05,.22,.17)}
 else if(hs==2){const h=new THREE.Mesh(gSph,top);h.scale.set(.1,.07,.105);h.position.set(-.01,1.69,0);G.add(h);bx(G,top,.11,1.67,0,.1,.015,.12)}
 if(Math.random()<.35)bx(G,mat('#111111'),.095,1.64,0,.02,.032,.17);
 if(Math.random()<.25)bx(G,mat(pk(['#3a5a8a','#8a3a3a','#3a3a3a','#e0a030'])),-.13,1.25,0,.1,.3,.28);
 G.scale.setScalar(7.5*(.93+Math.random()*.12));return G}
function addSpiralCoins(){for(let ph=1;ph<MPH;ph+=.85){const th=MT0+ph,r=MR0-MB*ph+R(-12,12);addCoin(MX+r*Math.cos(th),MZ+r*Math.sin(th),100,true)}
 for(let q=0;q<6;q++){const a=q/6*PI*2;addCoin(MX+55*Math.cos(a),MZ+55*Math.sin(a),500,true)}}
function buildMountain(){
 const dm=c=>new THREE.MeshLambertMaterial({color:c,side:THREE.DoubleSide});
 const rib=(ph0,ph1,e0,e1,m,n)=>{const pos=[],pt=(ph,e)=>{const th=MT0+ph,r=MR0-MB*ph,h=mhf(ph),o=e(ph,r,h),rr=r+o[0];return[MX+rr*Math.cos(th),h+o[1],MZ+rr*Math.sin(th)]};
  for(let s=0;s<n;s++){const a=ph0+(ph1-ph0)*s/n,b=ph0+(ph1-ph0)*(s+1)/n,A0=pt(a,e0),A1=pt(a,e1),B0=pt(b,e0),B1=pt(b,e1);pos.push(...A0,...A1,...B0,...A1,...B1,...B0)}
  const g=new THREE.BufferGeometry();g.setAttribute('position',new THREE.Float32BufferAttribute(pos,3));g.computeVertexNormals();scene.add(new THREE.Mesh(g,m))};
 const E=(o,y)=>()=>[o,y],W2=2*PI*MB-MHW;
 rib(0,MPH,E(MHW,.05),E(-MHW,.05),dm('#4a505c'),1000);
 rib(0,MPH,E(.9,.12),E(-.9,.12),dm('#f4e9a8'),1000);
 for(const sg of[1,-1])rib(0,MPH,E(sg*(MHW+1),0),E(sg*(MHW+1),6),dm('#f2f2f2'),1000);
 rib(0,MP1,E(-MHW,0),ph=>[-W2,mhf(ph+2*PI)-mhf(ph)],dm('#9a8870'),800);
 rib(MP1,MPH,E(-MHW,0),(ph,r)=>[120-r,0],dm('#7fae6a'),400);
 rib(0,2*PI,E(MHW,0),(ph,r,h)=>[MHW+h*.9+.1,-h],dm('#7fae6a'),250);
 const hT=mhf(MP1),pl=new THREE.Mesh(new THREE.CircleGeometry(122,40),dm('#7fae6a'));pl.rotation.x=-PI/2;pl.position.set(MX,hT+.05,MZ);scene.add(pl);
 const tw=new THREE.Mesh(gCyl,mat('#e8e8e8'));tw.scale.set(7,90,7);tw.position.set(MX,hT+45,MZ);scene.add(tw);
 const ant=new THREE.Mesh(gCyl,mat('#cc3333'));ant.scale.set(1.4,50,1.4);ant.position.set(MX,hT+115,MZ);scene.add(ant);
 const cv2=document.createElement('canvas');cv2.width=1024;cv2.height=160;const x2=cv2.getContext('2d');x2.fillStyle='#ffffff';x2.font='bold 135px Arial,sans-serif';x2.textAlign='center';x2.fillText('HOLLYWOOD',512,132);
 const hs=new THREE.Mesh(new THREE.PlaneGeometry(300,47),new THREE.MeshBasicMaterial({map:new THREE.CanvasTexture(cv2),transparent:true,side:THREE.DoubleSide}));hs.position.set(MX-60,hT+40,MZ);hs.rotation.y=-PI/2;scene.add(hs);
 for(const dz of[-90,0,90]){const po=new THREE.Mesh(gBox,mat('#cccccc'));po.scale.set(2,30,2);po.position.set(MX-58,hT+15,MZ+dz);scene.add(po)}
 const L0=MX-SIZE+60,cx=(SIZE-60+MX)/2;
 for(const [w,y,c] of[[RW+16,.2,'#ece6dc'],[RW,.4,'#555b68'],[2.5,.6,'#f4e9a8']]){const m=new THREE.Mesh(new THREE.PlaneGeometry(L0,w),mat(c));m.rotation.x=-PI/2;m.position.set(cx,y,MZC);scene.add(m)}}
const MM={cy:120,R:100};
function drawMini(){
 const R=Math.max(70,Math.min(105,W*.1)),cx=W-R-16,cy=R+16,sc=R/950,rot=-PI/2-p.a;MM.cy=cy;MM.R=R;
 g.save();g.shadowBlur=0;g.beginPath();g.arc(cx,cy,R,0,PI*2);g.fillStyle='rgba(20,28,38,.85)';g.fill();g.clip();
 g.translate(cx,cy);g.rotate(rot);g.scale(sc,sc);g.translate(-p.x,-p.y);
 const ex=1000,x0=p.x-ex,x1=p.x+ex,y0=p.y-ex,y1=p.y+ex;
 const i0=Math.max(0,Math.floor(x0/CELL)),i1=Math.min(N-1,Math.floor(x1/CELL)),j0=Math.max(0,Math.floor(y0/CELL)),j1=Math.min(N-1,Math.floor(y1/CELL));
 for(let i=i0;i<=i1;i++)for(let j=j0;j<=j1;j++){if(isWater(i,j))continue;g.fillStyle=blockColor(i,j);g.fillRect(i*CELL+RW+12,j*CELL+RW+12,276,276)}
 g.strokeStyle='#8a90a0';g.lineWidth=RW;g.beginPath();
 for(let j=j0;j<=j1;j++){g.moveTo(Math.max(0,x0),j*CELL+50);g.lineTo(Math.min(SIZE,x1),j*CELL+50)}
 for(let i=i0;i<=i1;i++){g.moveTo(i*CELL+50,Math.max(0,y0));g.lineTo(i*CELL+50,Math.min(SIZE,y1))}
 g.moveTo(SIZE,MZC);g.lineTo(MX,MZC);g.stroke();
 g.fillStyle='#3d7fb0';for(const j of RR)g.fillRect(Math.max(0,x0),j*CELL+RW,Math.min(SIZE,x1)-Math.max(0,x0),300);for(const i of RC)g.fillRect(i*CELL+RW,Math.max(0,y0),300,Math.min(SIZE,y1)-Math.max(0,y0));
 g.fillStyle='#d9b96a';for(const b of BR){if(b.h)g.fillRect(b.u0,b.v0,300,b.v1-b.v0);else g.fillRect(b.v0,b.u0,b.v1-b.v0,300)}
 g.fillStyle='#6b5a46';g.beginPath();g.arc(MX,MZ,745,0,PI*2);g.fill();
 g.strokeStyle='#e8cf7a';g.lineWidth=80;g.lineJoin='round';g.beginPath();
 for(let ph=0;ph<=MPH;ph+=.12){const th=MT0+ph,r=MR0-MB*ph,x=MX+r*Math.cos(th),y=MZ+r*Math.sin(th);if(ph==0)g.moveTo(x,y);else g.lineTo(x,y)}g.stroke();
 g.fillStyle='#f4f4f4';g.beginPath();g.arc(MX,MZ,125,0,PI*2);g.fill();
 g.fillStyle='#ffd84a';for(const c of coins){if(Math.abs(c.x-p.x)>ex||Math.abs(c.y-p.y)>ex)continue;g.beginPath();g.arc(c.x,c.y,c.val>=500?34:20,0,PI*2);g.fill()}
 const fl=Math.floor(Date.now()/200)%2;miniExtras(ex);
 if(POL.state=='search'){g.strokeStyle='#ffd86b';g.lineWidth=24;g.setLineDash([70,45]);g.beginPath();g.arc(POL.lx,POL.ly,280,0,PI*2);g.stroke();g.setLineDash([])}
 for(const c of cars){if(Math.abs(c.x-p.x)>ex||Math.abs(c.y-p.y)>ex)continue;
  if(c.t=='police'){if(!c.wreck&&POL.state!='chase'&&Math.hypot(c.x-p.x,c.y-p.y)>700)continue;g.fillStyle=c.wreck?'#555555':(fl?'#ff3b4d':'#4d7bff');g.beginPath();g.arc(c.x,c.y,c.wreck?22:46,0,PI*2);g.fill();if(!c.wreck){g.strokeStyle='#ffffff';g.lineWidth=12;g.stroke()}}
  else{g.fillStyle='#c9ced8';g.beginPath();g.arc(c.x,c.y,20,0,PI*2);g.fill()}}
 g.restore();
 g.save();g.translate(cx,cy);g.fillStyle='#ffffff';g.strokeStyle='#111111';g.lineWidth=2;g.beginPath();g.moveTo(0,-11);g.lineTo(8,9);g.lineTo(0,4);g.lineTo(-8,9);g.closePath();g.fill();g.stroke();g.restore();
 const c0=Math.cos(rot),s0=Math.sin(rot);
 for(const c of cars){if(c.t!='police'||c.wreck||(POL.state!='chase'&&Math.hypot(c.x-p.x,c.y-p.y)>700))continue;const dx=c.x-p.x,dy=c.y-p.y,rx=(dx*c0-dy*s0)*sc,ry=(dx*s0+dy*c0)*sc,d=Math.hypot(rx,ry);if(d<R-6)continue;
  const ux=rx/d,uy=ry/d,ax=cx+ux*(R-6),ay=cy+uy*(R-6);g.fillStyle=fl?'#ff3b4d':'#4d7bff';g.beginPath();g.moveTo(ax+ux*7,ay+uy*7);g.lineTo(ax-ux*5-uy*6,ay-uy*5+ux*6);g.lineTo(ax-ux*5+uy*6,ay-uy*5-ux*6);g.closePath();g.fill()}
 {const dx=MX-p.x,dy=MZ-p.y,rx=(dx*c0-dy*s0)*sc,ry=(dx*s0+dy*c0)*sc,d=Math.hypot(rx,ry);if(d>R-12){const ux=rx/d,uy=ry/d;g.fillStyle='#ffb347';g.font='bold 13px sans-serif';g.textAlign='center';g.textBaseline='middle';g.fillText('\u25B2\u5C71',cx+ux*(R-14),cy+uy*(R-14))}}
 g.strokeStyle='rgba(255,255,255,.85)';g.lineWidth=3;g.beginPath();g.arc(cx,cy,R,0,PI*2);g.stroke();
 g.fillStyle='#ff6b6b';g.font='bold 14px sans-serif';g.textAlign='center';g.textBaseline='middle';g.fillText('N',cx+Math.sin(rot)*(R-9),cy-Math.cos(rot)*(R-9));
 g.textBaseline='alphabetic';g.restore()}
function drawCompass(){
 const cx=W/2,w=Math.min(460,W*.46),y=12,h=28,psi=((Math.atan2(Math.cos(p.a),-Math.sin(p.a))*180/PI)+360)%360;
 const rel=b=>((b-psi+540)%360)-180,X=b=>cx+rel(b)/90*(w/2);
 g.save();g.shadowBlur=0;g.fillStyle='rgba(20,28,38,.75)';g.fillRect(cx-w/2,y,w,h);g.beginPath();g.rect(cx-w/2,y,w,h);g.clip();
 g.strokeStyle='rgba(255,255,255,.55)';g.lineWidth=1;g.font='bold 14px sans-serif';g.textAlign='center';g.textBaseline='middle';
 const LB={0:'N',45:'NE',90:'E',135:'SE',180:'S',225:'SW',270:'W',315:'NW'};
 for(let b=0;b<360;b+=15){const x=X(b);if(x<cx-w/2||x>cx+w/2)continue;g.beginPath();g.moveTo(x,y+h);g.lineTo(x,y+h-(b%45==0?9:5));g.stroke();const L=LB[b];if(L){g.fillStyle=b==0?'#ff6b6b':'#ffffff';g.fillText(L,x,y+11)}}
 for(const c of cars){if(c.t!='police'||c.wreck)continue;const b=Math.atan2(c.x-p.x,-(c.y-p.y))*180/PI,d=rel(b);if(Math.abs(d)>90)continue;const x=X(b);g.fillStyle=Math.floor(Date.now()/200)%2?'#ff3b4d':'#4d7bff';g.beginPath();g.moveTo(x,y+h-1);g.lineTo(x-4,y+h-9);g.lineTo(x+4,y+h-9);g.closePath();g.fill()}
 g.restore();
 g.save();g.shadowBlur=0;g.strokeStyle='rgba(255,255,255,.8)';g.lineWidth=2;g.strokeRect(cx-w/2,y,w,h);g.fillStyle='#ffd86b';g.beginPath();g.moveTo(cx,y+h+7);g.lineTo(cx-5,y+h+14);g.lineTo(cx+5,y+h+14);g.closePath();g.fill();
 const dx=MX-p.x,dy=MZ-p.y,bm=Math.atan2(dx,-dy)*180/PI,dm=rel(bm),dist=Math.hypot(dx,dy)/7.5,lab='\u5C71 '+(dist>=1000?(dist/1000).toFixed(1)+'km':Math.round(dist/10)*10+'m');
 g.font='bold 14px sans-serif';g.textAlign='center';g.textBaseline='middle';g.fillStyle='#ffb347';
 const mx=clamp(cx+dm/90*(w/2),cx-w/2+24,cx+w/2-24);
 g.fillText(Math.abs(dm)>90?(dm<0?'\u25C0 '+lab:lab+' \u25B6'):'\u25B2 '+lab,mx,y+h+26);
 g.fillStyle='#ffffff';g.fillText(Math.round(psi)+'\u00B0',cx,y+h+44);
 g.textBaseline='alphabetic';g.restore()}
const SIGSET=new Set(),SIGP=[],SGM={r:new THREE.MeshBasicMaterial({color:0xff2a2a}),y:new THREE.MeshBasicMaterial({color:0xffd23f}),g:new THREE.MeshBasicMaterial({color:0x2dff7a})};
const sigU=(i,j)=>((t||0)+hash(i*5+3,j*11+7)*16)%16;
function sigState(i,j,ax){const u=sigU(i,j);if(ax==0)return u>=.5&&u<6.5?'g':u>=6.5&&u<8?'y':'r';return u>=8.5&&u<14.5?'g':u>=14.5&&u<16?'y':'r'}
function sigRemain(i,j,ax){const u=sigU(i,j);if(ax==0)return u>=8?16.5-u:u<.5?.5-u:0;return u<8.5?8.5-u:0}
const sigGoPed=(i,j,ax)=>sigState(i,j,ax)=='r'&&sigRemain(i,j,ax)>3.5;
const asH=y=>(((y%CELL)+CELL)%CELL)<RW,asV=x=>(((x%CELL)+CELL)%CELL)<RW;
function pedCrossOK(q,nx,ny){
 if(asH(ny)&&!asH(q.y)){const i=Math.round((q.x-50)/CELL),j=Math.floor(ny/CELL);if(Math.abs(q.x-(i*CELL+50))>115)return false;return sigGoPed(i,j,0)}
 if(asV(nx)&&!asV(q.x)){const j=Math.round((q.y-50)/CELL),i=Math.floor(nx/CELL);if(Math.abs(q.y-(j*CELL+50))>115)return false;return sigGoPed(i,j,1)}
 return true}
function trafficSpeed(c){let tg=c.sp;const h=c.d%2==0,inc=c.d<2,al=h?c.x:c.y,m=((al%CELL)+CELL)%CELL;
 if(m>=RW){const dist=inc?CELL-m:m-RW,kk=inc?Math.floor(al/CELL)+1:Math.floor(al/CELL),oth=Math.round(((h?c.y:c.x)-50)/CELL),i=h?kk:oth,j=h?oth:kk;
  if(dist<110&&i>=0&&i<N&&j>=0&&j<N){const s=sigState(i,j,h?0:1);if(s=='r'||(s=='y'&&dist>40))tg=Math.min(tg,clamp((dist-24)/70,0,1)*c.sp)}}
 for(const o of cars){if(o===c||o.t!='t')continue;if(!o.free&&o.d!==c.d)continue;const lat=h?o.y-c.y:o.x-c.x;if(Math.abs(lat)>16)continue;
  const gap=(inc?1:-1)*(h?o.x-c.x:o.y-c.y);if(gap>0&&gap<90)tg=Math.min(tg,clamp((gap-38)/45,0,1)*c.sp)}
 return tg}
function updateSignals(){
 if(!SIGP.length)for(let q=0;q<120;q++){const m=new THREE.Mesh(gBox,SGM.r);m.scale.set(4.4,2.4,4.4);m.visible=false;scene.add(m);SIGP.push(m)}
 let n=0;const ci=Math.round((p.x-50)/CELL),cj=Math.round((p.y-50)/CELL);
 for(let i=ci-2;i<=ci+2;i++)for(let j=cj-2;j<=cj+2;j++){if(!SIGSET.has(i+','+j))continue;const X=i*CELL,Z=j*CELL,ns=sigState(i,j,1),ew=sigState(i,j,0);
  for(const [hx,hz,s] of[[X+32,Z-6,ns],[X-6,Z+32,ew],[X+68,Z+106,ns],[X+106,Z+68,ew]]){if(n>=SIGP.length)break;const m=SIGP[n++];m.visible=true;m.material=SGM[s];m.position.set(hx,s=='r'?23:s=='y'?20:17,hz)}}
 for(;n<SIGP.length;n++)SIGP[n].visible=false}
const TH=[0,25,65,120,190,280],CNT=[0,1,3,4,5,7],CD=[0,12,16,20,26,32],DN=['\u30a4\u30fc\u30b8\u30fc','\u30ce\u30fc\u30de\u30eb','\u30cf\u30fc\u30c9'],SIGHT=[330,430,520],PLAN=[1.4,.8,.4],FLANK=[3,2,1],CDM=[.65,1,1.4],AMUL=[.88,1,1.1],NB=[[1,0],[-1,0],[0,1],[0,-1]];
let stars=0,heat=0,bustT=0,money=0,notes=[],spawnT=0,flash=0,diff=1,endMsg='BUSTED!',recT=0,POL={state:'idle',lx:0,ly:0,lvx:0,lvy:0,lostT:0,searchT:0,seen:false,time:0,maxS:0};
function notify(txt,col,sub){notes.push({txt,col,sub:sub||'',t0:performance.now(),dur:2600});if(notes.length>3)notes.shift()}
function crime(h){heat=Math.min(400,heat+h)}
function los(ax,ay,bx,by){const d=Math.hypot(bx-ax,by-ay),n=Math.ceil(d/45);for(let i=1;i<n;i++){const x=ax+(bx-ax)*i/n,y=ay+(by-ay)*i/n,ci=Math.floor(x/CELL),cj=Math.floor(y/CELL);if(ci<0||cj<0||ci>=N||cj>=N)continue;if(isOpen(ci,cj)||isWater(ci,cj))continue;const lx=x-ci*CELL,ly=y-cj*CELL;if(ALLEYK[blockKind(ci,cj)]&&lx>RW+130&&lx<RW+170)continue;if(lx>RW+12&&lx<CELL-12&&ly>RW+12&&ly<CELL-12)return false}return true}
const nodeOf=(x,y)=>[clamp(Math.round((x-50)/CELL),0,N-1),clamp(Math.round((y-50)/CELL),0,N-1)];
function nearNode(x,y,a){const ci=Math.floor((x-50)/CELL),cj=Math.floor((y-50)/CELL),hx=Math.cos(a),hy=Math.sin(a);let best=null,bs=1e9;
 for(let i=ci;i<=ci+1;i++)for(let j=cj;j<=cj+1;j++){const ii=clamp(i,0,N-1),jj=clamp(j,0,N-1),dx=ii*CELL+50-x,dy=jj*CELL+50-y,d=Math.hypot(dx,dy),sc=d+((dx*hx+dy*hy)<0&&d>60?260:0);if(sc<bs){bs=sc;best=[ii,jj]}}
 return best}
function navPath(si,sj,ti,tj){const prev=new Int32Array(N*N).fill(-1),s0=si+sj*N,t0=ti+tj*N,q=[s0];prev[s0]=s0;
 for(let h=0;h<q.length;h++){const id=q[h];if(id==t0)break;const i=id%N,j=(id-i)/N;
  for(const [di,dj] of NB){const ni=i+di,nj=j+dj;if(ni<0||nj<0||ni>=N||nj>=N)continue;const nid=ni+nj*N;if(prev[nid]>=0)continue;
   if(segClosed(i*CELL+50,j*CELL+50,ni*CELL+50,nj*CELL+50))continue;prev[nid]=id;q.push(nid)}}
 if(prev[t0]<0)return null;const path=[];let id=t0;while(id!=s0){path.push([id%N,(id-id%N)/N]);id=prev[id]}path.push([si,sj]);path.reverse();return path}
function navTo(c,gx,gy){const sn=nearNode(c.x,c.y,c.a),tn=nodeOf(gx,gy),key=sn[0]+','+sn[1]+'>'+tn[0]+','+tn[1];
 if(c.pk!==key||c.pt===undefined||c.pt<=0){c.pk=key;c.pt=PLAN[diff]*60;c.path=navPath(sn[0],sn[1],tn[0],tn[1])}
 c.pt-=1;const path=c.path;if(!path)return{x:gx,y:gy};
 let w=path[0];if(path.length>1&&Math.hypot(w[0]*CELL+50-c.x,w[1]*CELL+50-c.y)<95)w=path[1];
 if(path.length<=2&&Math.hypot(gx-c.x,gy-c.y)<CELL*.9)return{x:gx,y:gy};
 return{x:w[0]*CELL+50,y:w[1]*CELL+50}}
function searchGoal(c){
 if(!c.sg||Math.hypot(c.sg.x-c.x,c.sg.y-c.y)<110){const first=!c.sgDone;c.sgDone=true;if(c.sg)c.hold=R(30,90);
  const ln=nodeOf(POL.lx+POL.lvx*40,POL.ly+POL.lvy*40);let ni=ln[0],nj=ln[1];
  if(!first){ni=clamp(ni+Math.floor(R(-3,4)),0,N-1);nj=clamp(nj+Math.floor(R(-3,4)),0,N-1)}
  c.sg={x:ni*CELL+50,y:nj*CELL+50}}
 return navTo(c,c.sg.x,c.sg.y)}
function policeGoal(c,k){
 if(c.leave)return{x:c.x+(c.x-p.x)*4,y:c.y+(c.y-p.y)*4};
 const dp=Math.hypot(p.x-c.x,p.y-c.y),mt=p.x>SIZE-150||mInfo(p.x,p.y);
 if(mt){const gx=(N-1)*CELL+50,gy=(N>>1)*CELL+50;if(c.x<SIZE&&Math.hypot(c.x-gx,c.y-gy)>150)return navTo(c,gx,gy);return policeTarget(c)}
 if(POL.state=='search')return searchGoal(c);
 c.sg=null;c.sgDone=false;
 if(c.sees&&dp<300)return{x:p.x+p.vx*8,y:p.y+p.vy*8};
 const r=c.role||0;let gx=p.x,gy=p.y;
 if(r>0&&stars>=FLANK[diff]){const sp=Math.hypot(p.vx,p.vy),hx=sp>.5?p.vx/sp:Math.cos(p.a),hy=sp>.5?p.vy/sp:Math.sin(p.a),L=r==1?clamp(sp*70,220,600):r==2?clamp(sp*110,380,900):300,sd=r%2?1:-1;
  gx=clamp(p.x+hx*L+(r>=3?-hy*sd*320:0),100,SIZE-100);gy=clamp(p.y+hy*L+(r>=3?hx*sd*320:0),100,SIZE-100);
  if(Math.hypot(gx-c.x,gy-c.y)<150)return{x:p.x,y:p.y}}
 return navTo(c,gx,gy)}
function spawnCop(a0=700,a1=1000){const hx=Math.cos(p.a),hy=Math.sin(p.a);
 for(let tr=0;tr<14;tr++){const s=spawnPt(a0,a1),dx=s.x-p.x,dy=s.y-p.y,d=Math.hypot(dx,dy);
  if(d<a0*.8)continue;if(tr<12&&(dx*hx+dy*hy)/d>.3)continue;if(tr<12&&d<650&&los(s.x,s.y,p.x,p.y))continue;
  const c=mkCar(s.x,s.y,Math.atan2(p.y-s.y,p.x-s.x),'police','#fff');c.acc=(.062+.005*stars)*.7*AMUL[diff];cars.push(c);return}}
function starUp(pr){notify('\u2605'.repeat(stars)+' \u624b\u914d\u30ec\u30d9\u30eb '+stars,'#ff6b6b',pr==0?'\u8b66\u5bdf\u306b\u8ffd\u308f\u308c\u3066\u3044\u308b\uff01':'\u8b66\u5bdf\u304c\u5897\u63f4\u3092\u547c\u3093\u3060\uff01');flash=1}
function lowerStar(){stars--;heat=stars>0?TH[stars]:0;POL.searchT=0;
 if(stars>0){notify('\u624b\u914d\u30ec\u30d9\u30eb\u304c\u4e0b\u304c\u3063\u305f','#ffd86b','\u2605'+stars)}
 else{const r=Math.round((250+POL.time*6)*(1+POL.maxS*.6));money+=r;score+=r;notify('\u9003\u8d70\u6210\u529f\uff01','#6be08a','\u5831\u916c $'+r);POL.state='idle';POL.time=0;POL.maxS=0}}
function endGame(msg){if(state!='play')return;state='over';endMsg=msg;burst(p.x,p.y,'#ff9a4a',30);const tot=Math.floor(score+t*10);if(tot>best)best=tot}
function recoverPlayer(){if(recT>0||mInfo(p.x,p.y))return;const i=clamp(Math.round((p.x-50)/CELL),1,N-2),j=clamp(Math.round((p.y-50)/CELL),1,N-2);
 p.x=i*CELL+50;p.y=j*CELL+50;p.a=Math.round(p.a/(PI/2))*(PI/2);p.vx=p.vy=0;inv=30;recT=300;notify('\u8eca\u3092\u5fa9\u5e30\u3057\u307e\u3057\u305f','#cfd8dc')}
function polUpdate(dt,k){
 if(recT>0)recT-=k;
 const act=[];for(const c of cars)if(c.t=='police'&&!c.wreck)act.push(c);
 const live=act.filter(c=>!c.leave).sort((a,b)=>Math.hypot(a.x-p.x,a.y-p.y)-Math.hypot(b.x-p.x,b.y-p.y));live.forEach((c,i)=>c.role=i);
 let seen=false;for(const c of live){c.sees=Math.hypot(c.x-p.x,c.y-p.y)<SIGHT[diff]&&los(c.x,c.y,p.x,p.y);if(c.sees)seen=true}
 POL.seen=seen;
 if(stars>0){
  if(seen){if(POL.state=='search')notify('\u898b\u3064\u304b\u3063\u305f\uff01','#ff6b6b');POL.state='chase';POL.lx=p.x;POL.ly=p.y;POL.lvx=p.vx;POL.lvy=p.vy;POL.lostT=0;POL.searchT=0;POL.time+=dt;if(stars<5)heat+=dt*.9}
  else{POL.lostT+=dt;if(POL.state=='chase'&&POL.lostT>2.5){POL.state='search';notify('\u8b66\u5bdf\u304c\u635c\u7d22\u4e2d','#ffd86b','\u898b\u3064\u304b\u3089\u306a\u3044\u3088\u3046\u306b\u9003\u3052\u7d9a\u3051\u3088\u3046')}
   if(POL.state=='search'){POL.searchT+=dt;if(POL.searchT>=CD[stars]*CDM[diff])lowerStar()}}}
 else{heat=Math.max(0,heat-dt*1.5);POL.state='idle'}
 let sN=0;for(let i=1;i<=5;i++)if(heat>=TH[i])sN=i;
 if(sN>stars){const pr=stars;stars=sN;POL.maxS=Math.max(POL.maxS,sN);POL.state='chase';POL.lostT=0;POL.searchT=0;if(pr==0){POL.lx=p.x;POL.ly=p.y;POL.time=0}starUp(pr)}
 const want=CNT[stars];
 if(stars==0){for(const c of act)c.leave=true}
 else{for(let i=want;i<live.length;i++)live[i].leave=true;spawnT-=k;if(live.length<want&&spawnT<=0&&POL.state!='search'){spawnCop(700,1000);spawnT=70}}
 for(let i=cars.length-1;i>=0;i--){const c=cars[i];if(c.t=='police'&&c.leave&&!c.wreck){const d=Math.hypot(c.x-p.x,c.y-p.y);if(d>520||(d>250&&!los(c.x,c.y,p.x,p.y))){kill(c);cars.splice(i,1)}}}
 const near=live.filter(c=>Math.hypot(c.x-p.x,c.y-p.y)<62).length,ps=Math.hypot(p.vx,p.vy);
 if(stars>0&&ps<.9&&near>0)bustT+=dt*(near>1?1:.6);else bustT=Math.max(0,bustT-dt*1.5);
 if(bustT>3.5)endGame('BUSTED!')}
function drawNotes(){const now=performance.now();notes=notes.filter(n=>now-n.t0<n.dur);let y=H*.26;
 if(flash>0){g.save();g.strokeStyle='rgba(255,60,60,'+(flash*.7)+')';g.lineWidth=18;g.strokeRect(0,0,W,H);g.restore();flash=Math.max(0,flash-.02)}
 for(const n of notes){const a=now-n.t0,sc=1+.4*Math.max(0,1-a/220),al=clamp((n.dur-a)/500,0,1);g.save();g.globalAlpha=al;g.translate(W/2,y);g.scale(sc,sc);g.textAlign='center';g.shadowColor='#000';g.shadowBlur=8;g.fillStyle=n.col;g.font='bold 40px sans-serif';g.fillText(n.txt,0,0);if(n.sub){g.font='bold 18px sans-serif';g.fillStyle='#fff';g.fillText(n.sub,0,28)}g.restore();y+=74}}
function drawHud2(){g.save();g.shadowBlur=4;g.shadowColor='#000';g.textAlign='left';g.font='bold 18px sans-serif';g.fillStyle='#8be28b';g.fillText('$ '+money,16,140);
 const left=Math.max(0,Math.ceil(CD[stars]*CDM[diff]-POL.searchT)),st=stars==0?['\u624b\u914d\u306a\u3057','#cfd8dc']:POL.state=='search'?['\u635c\u7d22\u4e2d '+left+'s','#ffd86b']:['\u8ffd\u8de1\u4e2d','#ff6b6b'];
 g.fillStyle=st[1];g.fillText(st[0],16,164);
 g.textAlign='center';g.font='bold 42px sans-serif';g.fillStyle='#fff';g.fillText(Math.round(Math.hypot(p.vx,p.vy)*28.8),W/2,H-44);g.font='bold 14px sans-serif';g.fillText('km/h',W/2,H-24);
 if(bustT>0){const w=Math.min(260,W*.4);g.fillStyle='rgba(0,0,0,.55)';g.fillRect(W/2-w/2,H*.72,w,14);g.fillStyle='#ff4d5e';g.fillRect(W/2-w/2,H*.72,w*Math.min(1,bustT/3.5),14);g.fillStyle='#fff';g.font='bold 14px sans-serif';g.fillText('\u9022\u6355\u30e1\u30fc\u30bf\u30fc\uff08\u52d5\u3044\u3066\u8105\u51fa\uff01\uff09',W/2,H*.72-6)}
 g.restore();drawNotes()}
function pedMesh2(col){const G=new THREE.Group(),pk=a=>a[Math.floor(Math.random()*a.length)],sk=mat(pk(SK)),pm=mat(pk(PANT));
 const limb=(m,z,len,w,y)=>{const piv=new THREE.Group();piv.position.set(0,y,z);bx(piv,m,0,-len/2,0,w,len,w);G.add(piv);return piv};
 G.lL=limb(pm,-1.3,7,2.4,7.2);G.lR=limb(pm,1.3,7,2.4,7.2);
 const sl=Math.random()<.5?mat(col):sk;G.aL=limb(sl,-3.4,5.6,1.7,12.6);G.aR=limb(sl,3.4,5.6,1.7,12.6);
 bx(G,mat(col),0,10,0,3.2,6,5.2);
 const hd=new THREE.Mesh(gSph,sk);hd.scale.set(2.2,2.5,2.2);hd.position.y=15.4;G.add(hd);
 const hr=new THREE.Mesh(gSph,mat(pk(HR)));hr.scale.set(2.35,1.7,2.35);hr.position.set(-.2,16.4,0);G.add(hr);
 G.scale.setScalar(.78);return G}
const KINDS=[{x0:-12,x1:8,x2:-7,x3:2,top:17.5,bh:11},{x0:-16,x1:9,x2:-14,x3:4,top:20,bh:12},{x0:-10,x1:6,x2:-5,x3:0,top:16,bh:10.5},{x0:-15,x1:7,x2:-12,x3:2,top:18.5,bh:11}];
const ext=(pts,w)=>{const g=new THREE.ExtrudeGeometry(new THREE.Shape(pts.map(a=>new THREE.Vector2(a[0],a[1]))),{depth:w,bevelEnabled:false});g.translate(0,0,-w/2);return g};
KINDS.forEach(k=>{const b=k.bh;
 k.body=ext([[-17,3],[17,3],[17.6,6.5],[16.5,b-1.2],[9,b-.2],[-14,b],[-17,b-1.5]],18);
 k.cab=ext([[k.x0,b],[k.x1,b],[k.x3,k.top],[k.x2,k.top]],15.6);
 k.end=ext([[k.x0-.4,b+.3],[k.x1+.4,b+.3],[k.x3+.4,k.top-.3],[k.x2-.4,k.top-.3]],13.4);
 k.side=ext([[k.x0+1.2,b+.6],[k.x1-1.2,b+.6],[k.x3-.6,k.top-.8],[k.x2+.6,k.top-.8]],15.9)});
const SHM=new THREE.MeshBasicMaterial({color:0x000000,transparent:true,opacity:.28,depthWrite:false});
function carMesh2(type,col,kind,opt){if(kind===4)return truckMesh(col);const G=new THREE.Group(),k=KINDS[kind!==undefined?kind:(type=='p'?2:type=='police'?0:Math.floor(Math.random()*4))],m=mat(col),gl=mat('#1c2733'),dk=mat('#222222'),b=k.bh;
 const mk=(geo,mm)=>{const o=new THREE.Mesh(geo,mm);G.add(o);return o};
 const body=mk(k.body,m),cab=mk(k.cab,m);mk(k.end,gl);mk(k.side,gl);
 for(const sx of[-10.5,11])for(const sz of[-8.6,8.6]){const w=new THREE.Mesh(gCyl,dk);w.rotation.x=PI/2;w.scale.set(3.8,2.6,3.8);w.position.set(sx,3.8,sz);G.add(w);
  const h=new THREE.Mesh(gCyl,mat('#bbbbbb'));h.rotation.x=PI/2;h.scale.set(2,2.8,2);h.position.set(sx,3.8,sz);G.add(h)}
 for(const z of[-5.8,5.8]){bx(G,mat('#fff6c8'),17.5,b-3.2,z,.8,1.8,3.4);bx(G,mat('#ff2a2a'),-17.5,b-3.2,z,.8,1.8,3.4)}
 bx(G,dk,17.6,4.6,0,1.2,2,17.6);bx(G,dk,-17.6,4.6,0,1.2,2,17.6);bx(G,dk,17.7,b-4.2,0,.6,1.6,6);bx(G,mat('#f5f5f5'),-17.8,5,0,.4,2,6);
 for(const z of[-9.4,9.4])bx(G,m,k.x1-1,b+1.4,z,1.8,1.4,1.6);
 if(type=='p'&&k===KINDS[2]){bx(G,m,-16.5,b+3.5,0,3.4,.8,16);for(const z of[-6,6])bx(G,dk,-16.5,b+1.8,z,.8,3,.8);bx(G,mat('#ffffff'),2,b+.05,0,22,.2,4)}
 if(type=='police'){bx(G,dk,-1,7,0,12,3.4,18.3);bx(G,dk,13,b-.3,0,6,.3,16);const xr=(k.x2+k.x3)/2;G.l1=bx(G,LR,xr,k.top+1,-3.6,3,1.6,6);G.l2=bx(G,LB,xr,k.top+1,3.6,3,1.6,6)}
 if(opt=='taxi'){bx(G,mat('#fffbe0'),-2,k.top+2,0,6,2.5,10);bx(G,mat('#222222'),0,6,0,8,1.2,18.4)}G.rotation.order='YZX';G.tint=[body,cab];const s=new THREE.Mesh(new THREE.PlaneGeometry(42,24),SHM);s.rotation.x=-PI/2;s.position.y=.5;G.add(s);return G}
const tintCar=(c,col)=>c.mesh.tint.forEach(o=>o.material=mat(col));
const kill=e=>scene.remove(e.mesh);
/* ---------- world ---------- */
const M4=new THREE.Matrix4(),Q=new THREE.Quaternion(),S=new THREE.Vector3(),P=new THREE.Vector3(),C=new THREE.Color();
const INST=[];
function inst(geo,m){const w={geo,m,ch:new Map()};INST.push(w);return w}
function add(w,x,y,z,sx,sy,sz,col,q){M4.compose(P.set(x,y,z),q||Q,S.set(sx,sy,sz));const kx=Math.floor(x/800),kz=Math.floor(z/800),key=kx+','+kz;let c=w.ch.get(key);if(!c){c={m:[],c:[],x:kx*800+400,z:kz*800+400};w.ch.set(key,c)}
 const e=M4.elements;for(let i=0;i<16;i++)c.m.push(e[i]);if(col)c.c.push(col.r,col.g,col.b);else c.c.push(1,1,1)}
function finInst(){for(const w of INST)for(const c of w.ch.values()){const n=c.m.length/16,g=w.geo.clone();g.boundingSphere=new THREE.Sphere(new THREE.Vector3(c.x,200,c.z),1500);
  const o=new THREE.InstancedMesh(g,w.m,n);o.instanceMatrix.array.set(c.m);o.instanceColor=new THREE.BufferAttribute(new Float32Array(c.c),3);scene.add(o)}INST.length=0}
function winTex(){const c=document.createElement('canvas');c.width=c.height=128;const x=c.getContext('2d');x.fillStyle='#fff';x.fillRect(0,0,128,128);x.fillStyle='#9fc4e0';
 for(let i=0;i<8;i++)for(let j=0;j<8;j++)x.fillRect(3+i*16,4+j*16,9,9);return new THREE.CanvasTexture(c)}
function build(){
 const gr=new THREE.Mesh(new THREE.PlaneGeometry(SIZE+5000,SIZE+5000),mat('#cfe8c4'));gr.rotation.x=-PI/2;gr.position.set(SIZE/2,0,SIZE/2);scene.add(gr);
 for(let j=0;j<0;j++)for(const [m,w,y] of [[mat('#ece6dc'),RW+16,.2],[mat('#555b68'),RW,.4]]){
  const a=new THREE.Mesh(new THREE.PlaneGeometry(SIZE,w),m);a.rotation.x=-PI/2;a.position.set(SIZE/2,y,j*CELL+50);scene.add(a);
  const b=new THREE.Mesh(new THREE.PlaneGeometry(w,SIZE),m);b.rotation.x=-PI/2;b.position.set(j*CELL+50,y,SIZE/2);scene.add(b)}
 const ivs=bands=>{const o=[];let s=0;for(const [a,b] of bands){o.push([s,a]);s=b}o.push([s,SIZE]);return o};
 for(let a=0;a<N;a++){const hb=RC.map(i=>[i*CELL+RW,(i+1)*CELL]),vb=RR.map(j=>[j*CELL+RW,(j+1)*CELL]);
  for(const [m,w,y] of [[mat('#ece6dc'),RW+16,.2],[mat('#555b68'),RW,.4]]){
   for(const [s,e] of ivs(hb)){const q=new THREE.Mesh(new THREE.PlaneGeometry(e-s,w),m);q.rotation.x=-PI/2;q.position.set((s+e)/2,y,a*CELL+50);scene.add(q)}
   for(const [s,e] of ivs(vb)){const q=new THREE.Mesh(new THREE.PlaneGeometry(w,e-s),m);q.rotation.x=-PI/2;q.position.set(a*CELL+50,y,(s+e)/2);scene.add(q)}}}
 const dash=inst(gBox,mat('#f4e9a8'),11000);
 for(let j=0;j<N;j++)for(let s=0;s<SIZE;s+=44){if(!bandH(s+12))add(dash,s+12,.6,j*CELL+50,24,.2,3);if(!bandV(s+12))add(dash,j*CELL+50,.6,s+12,3,.2,24)}
 const bm=new THREE.MeshLambertMaterial({map:winTex()});BM=bm;
 const gCone=new THREE.ConeGeometry(1,1,4);gCone.rotateY(PI/4);
 const bases=inst(gBox,mat('#fff'),2000),bld=inst(gBox,bm,9000),roofs=inst(gBox,mat('#fff'),9000),trees=inst(gSph,mat('#fff'),6000),trunks=inst(gCyl,mat('#fff'),6000),cone=inst(gCone,mat('#fff'),6000),rl=inst(gBox,mat('#fff'),3500),cw=inst(gBox,mat('#fff'),12000),pond=inst(gCyl,mat('#fff'),500);
 const deck=inst(gBox,mat('#fff'),3800),pier=inst(gCyl,mat('#fff'),300),QE=new THREE.Euler(),QQ=new THREE.Quaternion();
 const dk=(x,y,z,sx,sy,sz,yaw,tilt,col)=>{QE.set(0,yaw,tilt,'YZX');QQ.setFromEuler(QE);add(deck,x,y,z,sx,sy,sz,C.set(col),QQ)};
 for(const b of BR){const NS=30,ds=10,yaw=b.h?0:-PI/2,vc=(b.v0+b.v1)/2,W2=(u,v)=>b.h?[u,vc+v]:[vc+v,u],hf=t=>ARCH*4*t*(1-t);
  for(let k=0;k<NS;k++){const h0=hf(k/NS),h1=hf((k+1)/NS),hm=(h0+h1)/2,um=b.u0+(k+.5)*ds,tl=Math.atan2(h1-h0,ds),L=ds+.5;
   const put=(v,y,sx,sy,sz,col)=>{const p=W2(um,v);dk(p[0],hm+y,p[1],sx,sy,sz,yaw,tl,col)};
   put(0,.15,L,.5,100,'#555b68');put(0,-1.6,L,3,116,'#b9b4a8');
   for(const s of[-1,1]){put(s*54,.45,L,.9,8,'#ddd8cc');put(s*58.5,4.4,L,2,1.4,'#f2f2f2');if(k%2==0)put(s*58.5,3,2,6,2,'#f2f2f2')}
   if(k%2==0)put(0,.55,ds*.55,.12,2.6,'#f4e9a8')}
  for(const u of[60,150,240])for(const s of[-40,40]){const hb=hf(u/300)-2.7,p=W2(b.u0+u,s);add(pier,p[0],hb/2,p[1],5,hb,5,C.set('#a8a398'))}}
 for(let a=0;a<N;a++){for(const j of RR)if(!BV.includes(a)){const x=a*CELL+50;add(rl,x,4,j*CELL+RW+3,RW+16,5,2.5,C.set('#e8553d'));add(rl,x,4,(j+1)*CELL-3,RW+16,5,2.5,C.set('#e8553d'))}
  for(const i of RC)if(!BH.includes(a)){const z=a*CELL+50;add(rl,i*CELL+RW+3,4,z,2.5,5,RW+16,C.set('#e8553d'));add(rl,(i+1)*CELL-3,4,z,2.5,5,RW+16,C.set('#e8553d'))}}
 const rails2=(i,j,x0,z0)=>{const WC='#f2f2f2',rr=RR.includes(j),cc=RC.includes(i);
  if(!cc){add(rl,x0+150,4,z0+5,300,2,1.5,C.set(WC));add(rl,x0+150,4,z0+295,300,2,1.5,C.set(WC));for(let q=0;q<=6;q++){add(rl,x0+q*50,3,z0+5,2,6,2,C.set(WC));add(rl,x0+q*50,3,z0+295,2,6,2,C.set(WC))}}
  if(!rr){add(rl,x0+5,4,z0+150,1.5,2,300,C.set(WC));add(rl,x0+295,4,z0+150,1.5,2,300,C.set(WC));for(let q=0;q<=6;q++){add(rl,x0+5,3,z0+q*50,2,6,2,C.set(WC));add(rl,x0+295,3,z0+q*50,2,6,2,C.set(WC))}}};
 const WM=mat('#6fc3ea');
 for(const a of[5,17]){const m=new THREE.Mesh(new THREE.PlaneGeometry(SIZE,300),WM);m.rotation.x=-PI/2;m.position.set(SIZE/2,.15,a*CELL+RW+150);scene.add(m)}
 for(const a of[9,19]){const m=new THREE.Mesh(new THREE.PlaneGeometry(300,SIZE),WM);m.rotation.x=-PI/2;m.position.set(a*CELL+RW+150,.15,SIZE/2);scene.add(m)}
 for(let i=0;i<N;i++)for(let j=0;j<N;j++)for(let q=0;q<5;q++){const o=14+q*18,X=i*CELL,Z=j*CELL;
  if(!bandH(X-8))add(cw,X-8,.65,Z+o,10,.1,6,C.set('#f4f4f4'));if(!bandH(X+RW+8))add(cw,X+RW+8,.65,Z+o,10,.1,6,C.set('#f4f4f4'));if(!bandV(Z-8))add(cw,X+o,.65,Z-8,6,.1,10,C.set('#f4f4f4'));if(!bandV(Z+RW+8))add(cw,X+o,.65,Z+RW+8,6,.1,10,C.set('#f4f4f4'))}
 const rails=(x0,z0)=>{const WC='#f2f2f2';
  add(rl,x0+150,4,z0+5,300,2,1.5,C.set(WC));add(rl,x0+150,4,z0+295,300,2,1.5,C.set(WC));add(rl,x0+5,4,z0+150,1.5,2,300,C.set(WC));add(rl,x0+295,4,z0+150,1.5,2,300,C.set(WC));
  for(let q=0;q<=6;q++){const o=q*50;add(rl,x0+o,3,z0+5,2,6,2,C.set(WC));add(rl,x0+o,3,z0+295,2,6,2,C.set(WC));add(rl,x0+5,3,z0+o,2,6,2,C.set(WC));add(rl,x0+295,3,z0+o,2,6,2,C.set(WC))}};
 const park=(i,j,x0,z0)=>{add(bases,x0+150,1.2,z0+150,300,.4,22,C.set('#efe3c8'));add(bases,x0+150,1.2,z0+150,22,.4,300,C.set('#efe3c8'));add(pond,x0+150,1.5,z0+150,48,.6,38,C.set('#6fc3ea'))};
 const lamp=inst(gCyl,mat('#fff'),3000),lampH=inst(gSph,new THREE.MeshBasicMaterial({color:0xffffff}),3000),hed=inst(gBox,mat('#fff'),4000);
 const lampAt=(x,z)=>{add(lamp,x,9,z,.9,18,.9,C.set('#4a4f58'));add(lampH,x,19,z,2,1.4,2,C.set('#fff3b0'))};
 for(let q=0;q<48;q++){const a=q/48*PI*2,d=SIZE*.5+1100+R(0,500);if(Math.cos(a)>.3&&Math.abs(Math.sin(a))<.75)continue;add(trees,SIZE/2+Math.cos(a)*d,0,SIZE/2+Math.sin(a)*d,R(500,800),R(250,500),R(500,800),C.set(['#a9cfa0','#9cc2b4','#b5d3a8'][q%3]))}
 const hedge=(x0,z0,col)=>{add(hed,x0+150,5,z0+14.5,276,9,5,C.set(col));add(hed,x0+150,5,z0+285.5,276,9,5,C.set(col));add(hed,x0+14.5,5,z0+150,5,9,276,C.set(col));add(hed,x0+285.5,5,z0+150,5,9,276,C.set(col))};
 const park2=(i,j,x0,z0)=>{const cx=x0+150,cz=z0+150;
  add(bases,cx,.67,cz,300,.1,22,C.set('#efe3c8'));add(bases,cx,.67,cz,22,.1,300,C.set('#efe3c8'));
  add(pond,cx,.68,cz,54,.25,44,C.set('#f4f4f4'));add(pond,cx,.72,cz,48,.25,38,C.set('#6fc3ea'));add(pond,cx,2.5,cz,6,4,6,C.set('#f4f4f4'));
  for(const s of[-1,1]){add(roofs,cx+s*85,2.5,cz+16,14,2,4,C.set('#8a5a3a'));add(roofs,cx+16,2.5,cz+s*85,4,2,14,C.set('#8a5a3a'))}
  add(roofs,x0+45,5,z0+45,8,10,8,C.set('#e0524a'));add(roofs,x0+62,3,z0+45,22,1.5,8,C.set('#e8a33d'));add(roofs,x0+45,2,z0+66,14,1,14,C.set('#3a86c8'));
  lampAt(cx+22,cz+22);lampAt(cx-22,cz-22);lampAt(x0+6,z0+150);lampAt(x0+294,z0+150)};
 const fenceLine=(lo,hi,gaps,cb)=>{let s=lo;for(const g of gaps){if(g-11>s)cb(s,g-11);s=g+11}if(hi>s)cb(s,hi)};
 const hedge2=(x0,z0,col,gxs,gzs)=>{
  fenceLine(x0+12,x0+288,gxs,(s,e)=>{add(hed,(s+e)/2,5,z0+14.5,e-s,9,5,C.set(col));add(hed,(s+e)/2,5,z0+285.5,e-s,9,5,C.set(col))});
  fenceLine(z0+12,z0+288,gzs,(s,e)=>{add(hed,x0+14.5,5,(s+e)/2,5,9,e-s,C.set(col));add(hed,x0+285.5,5,(s+e)/2,5,9,e-s,C.set(col))})};
 const resDoor=(x0,z0,a,b,x,z)=>{const hw=28;let D;
  if(b==0)D={x,y:z-hw+1,gx:x,gy:z0+6,bx:x,by:z-hw-.3,bw:9,bd:1.2};
  else if(b==2)D={x,y:z+hw-1,gx:x,gy:z0+294,bx:x,by:z+hw+.3,bw:9,bd:1.2};
  else if(a==0)D={x:x-hw+1,y:z,gx:x0+6,gy:z,bx:x-hw-.3,by:z,bw:1.2,bd:9};
  else if(a==2)D={x:x+hw-1,y:z,gx:x0+294,gy:z,bx:x+hw+.3,by:z,bw:1.2,bd:9};
  else return;
  doors.push(D);add(roofs,D.bx,5,D.by,D.bw,10,D.bd,C.set('#5a3d2b'))};
 const shopDoors=(x,z,w,sx,sz)=>{const h=w/2;
  doors.push({x:x+sx*(h-1),y:z,gx:x+sx*(h+6),gy:z});add(roofs,x+sx*(h+.3),5,z,1.2,10,9,C.set('#2b2b33'));
  doors.push({x:x,y:z+sz*(h-1),gx:x,gy:z+sz*(h+6)});add(roofs,x,5,z+sz*(h+.3),9,10,1.2,C.set('#2b2b33'))};
 const gPr=(()=>{const sh=new THREE.Shape([new THREE.Vector2(-.5,0),new THREE.Vector2(.5,0),new THREE.Vector2(0,1)]),g=new THREE.ExtrudeGeometry(sh,{depth:1,bevelEnabled:false});g.translate(0,0,-.5);g.rotateY(PI/2);return g})();
 const prism=inst(gPr),YV=new THREE.Vector3(0,1,0),TH4=[0,PI,-PI/2,PI/2],YQ=TH4.map(a=>new THREE.Quaternion().setFromAxisAngle(YV,a));
 const STU=['#efe6d2','#f4efe6','#e8d9b8','#d9c7a3','#dfe5e8','#e8cfc4','#cfdcc6','#f0e8d0','#e5ddc8'],ROOFC=['#b5542f','#a04a2c','#7a4a3a','#4d4d55','#6a5a4a'],LAWN=['#8fc97a','#a6d08a','#b9c98a','#c9c08a','#7fbf7a'],CARS=['#f2f2f2','#1f2328','#9ea4ab','#b3262b','#1f4e8c'],GLS=['#6f8fa8','#8aa6b8','#5f7c92','#a8b8c4','#7d93a5','#4e6a80','#c9d3da'],pk=a=>a[Math.floor(R(0,a.length))];
 const palm=(x,z,h)=>{add(trunks,x,h/2,z,1.4,h,1.4,C.set('#9a8460'));add(trees,x,h+1,z,9,3,9,C.set('#4f9a4a'));add(trees,x,h+3.5,z,5.5,2.4,5.5,C.set('#63b05a'))};
 const lowWall=(x0,z0,col,gxs,gzs)=>{fenceLine(x0+12,x0+288,gxs,(s,e)=>{add(hed,(s+e)/2,2.6,z0+13.5,e-s,4,3,C.set(col));add(hed,(s+e)/2,2.6,z0+286.5,e-s,4,3,C.set(col))});fenceLine(z0+12,z0+288,gzs,(s,e)=>{add(hed,x0+13.5,2.6,(s+e)/2,3,4,e-s,C.set(col));add(hed,x0+286.5,2.6,(s+e)/2,3,4,e-s,C.set(col))})};
 const QZ=new THREE.Quaternion().setFromAxisAngle(new THREE.Vector3(0,0,1),PI/2),QW=YQ.map(q=>q.clone().multiply(QZ)),CARS2=['#f2f2f2','#1f2328','#9ea4ab','#b3262b','#1f4e8c','#d9d9d9','#e4b43c','#2e6b4f','#7a2d2d'];
 const pcar=(x,z,fi,col)=>{const th=TH4[fi],cs=Math.cos(th),sn=Math.sin(th),q=YQ[fi],qw=QW[fi],wp=(lx,lz)=>[x+lx*cs+lz*sn,z-lx*sn+lz*cs];
  const P1=(w,lx,ly,lz,sx,sy,sz,c,qq)=>{const pp=wp(lx,lz);add(w,pp[0],ly,pp[1],sx,sy,sz,C.set(c),qq||q)};
  P1(roofs,0,4.3,0,15,5,29,col);P1(roofs,0,8.9,-2.5,13.4,4,15.5,col);P1(roofs,0,8.8,-2.5,13.9,2.6,16.2,'#1c2733');
  P1(roofs,0,5.6,14.8,11,1.4,.7,'#fff2c0');P1(roofs,0,5.6,-14.8,11,1.4,.7,'#ff2a2a');
  for(const sx of[-7.3,7.3])for(const sz of[-9.2,9.5])P1(trunks,sx,3.2,sz,3.2,2.4,3.2,'#1d1d1d',qw)};
 const lot=(x0,z0,col)=>add(bases,x0+150,.67,z0+150,276,.1,276,C.set(col));
 const carRow=(x,z,n,dx)=>{for(let q=0;q<n;q++){if(R(0,1)<.7)pcar(x+q*dx,z,Math.random()<.5?0:1,pk(CARS2));add(roofs,x+q*dx-dx/2,.8,z,.8,.2,32,C.set('#f4f4f4'))}};
 function house(x0,z0,x,z,fi,hasDoor){
  const th=TH4[fi],cs=Math.cos(th),sn=Math.sin(th),q=YQ[fi],wp=(lx,lz)=>[x+lx*cs+lz*sn,z-lx*sn+lz*cs];
  const B_=(w,lx,ly,lz,sx,sy,sz,col)=>{const p=wp(lx,lz);add(w,p[0],ly,p[1],sx,sy,sz,C.set(col),q)};
  const wall=pk(STU),rf=pk(ROOFC),st=Math.floor(R(0,4)),S0=-8;let dl=-10,dz=9;
  add(bases,x,.67,z,92,.1,92,C.set(pk(LAWN)));
  if(st==0){B_(bld,0,6.6,S0,50,12,34,wall);B_(cone,0,16,S0,38,8,26,rf);B_(bld,36,5.6,S0+2,22,10,30,wall);B_(cone,36,13,S0+2,17,6,23,rf);
   B_(roofs,36,.8,S0+32,18,.3,28,'#bdbab2');{const pc=wp(36,S0+32);pcar(pc[0],pc[1],fi,pk(CARS2))}dz=S0+17;dl=-10}
  else if(st==1){B_(bld,0,12.6,S0,46,24,36,'#f4efe6');B_(cone,0,29,S0,35,10,27,'#b5542f');B_(bld,-8,4.6,S0+23,22,8,10,'#f4efe6');B_(roofs,-8,9.2,S0+23,24,1.5,12,'#b5542f');
   if(R(0,1)<.5){B_(bld,16,17.6,S0-4,12,34,12,'#f4efe6');B_(cone,16,39,S0-4,9,10,9,'#b5542f')}dz=S0+28;dl=-8}
  else if(st==2){B_(bld,0,6.6,S0,56,12,36,'#f2f2f2');B_(bld,-6,17.6,S0-2,38,11,30,pk(['#e8e8e8','#cfd4d8','#b8a890']));B_(roofs,0,12.8,S0,59,1.5,39,'#3a3d44');B_(roofs,-6,23.8,S0-2,41,1.5,33,'#3a3d44');
   B_(roofs,-6,6,S0+18.4,26,9,.8,'#2a3a4a');if(R(0,1)<.6)B_(roofs,30,.85,S0+26,18,.3,24,'#4fc3e8');dz=S0+18;dl=-18}
  else{B_(bld,0,8.1,S0,42,15,32,pk(['#a8b8a0','#d9c7a3','#b8a890','#c9d3da']));B_(prism,0,15.6,S0,46,13,38,rf);B_(roofs,-8,1.2,S0+22,26,1,12,'#b9b5ac');B_(prism,-8,10,S0+22,28,6,14,rf);
   for(const sx of[-19,3])B_(roofs,sx,5,S0+27,1.6,10,1.6,'#f2f2f2');B_(roofs,14,22,S0,5,12,5,'#8a5a4a');dz=S0+16;dl=-8}
  const pp=wp(-33,S0+34);palm(pp[0],pp[1],R(30,44));
  B_(roofs,dl,5,dz+.5,8,10,1.2,'#5a3d2b');
  if(!hasDoor)return;
  const dp=wp(dl,dz-1);let gx,gy;
  if(fi==0){gx=dp[0];gy=z0+294}else if(fi==1){gx=dp[0];gy=z0+6}else if(fi==2){gx=x0+6;gy=dp[1]}else{gx=x0+294;gy=dp[1]}
  doors.push({x:dp[0],y:dp[1],gx,gy});return fi<2?gx:gy}
 const resi=(i,j,x0,z0)=>{const A=x0+12,B=z0+12,gx=[],gz=[],hOn=hash(i,j)<.35;
  for(let a=0;a<3;a++)for(let b=0;b<3;b++){const x=A+46+a*92,z=B+46+b*92,fi=b==2?0:b==0?1:a==0?2:a==2?3:0,g=house(x0,z0,x,z,fi,!(a==1&&b==1));if(g!==undefined)(fi<2?gx:gz).push(g)}
  gx.sort((u,v)=>u-v);gz.sort((u,v)=>u-v);
  if(hOn)hedge2(x0,z0,'#4f9d5a',gx,gz);else lowWall(x0,z0,'#d8d2c4',gx,gz)};
 const tower=(x,z,w,h,c)=>{const t2=h*.14,t3=h>200?h*.1:0,top=h+t2+t3+.6;add(bld,x,h/2+.6,z,w,h,w,C.set(c));add(bld,x,h+t2/2+.6,z,w*.72,t2,w*.72,C.set(c));if(t3)add(bld,x,h+t2+t3/2+.6,z,w*.45,t3,w*.45,C.set(c));
  add(roofs,x,top+1.5,z,w*.5,3,w*.5,C.set('#d5d8dc'));if(h>180)add(roofs,x,top+24,z,2.5,44,2.5,C.set('#c8ccd0'));
  add(trunks,x+w*.3,top+6,z-w*.3,5,9,5,C.set('#8a6a4a'));if(h>150&&hash(Math.floor(x),Math.floor(z))<.5){add(roofs,x-w*.1,top+14,z+w*.3,w*.5,14,1.5,C.set(pk(['#e63946','#ffd23f','#2a9d8f','#f4f4f4'])));add(roofs,x-w*.3,top+5,z+w*.3,1.5,10,1.5,C.set('#555555'));add(roofs,x+w*.1,top+5,z+w*.3,1.5,10,1.5,C.set('#555555'))}};
 const towers=(i,j,x0,z0,f)=>{for(let a=0;a<2;a++)for(let b=0;b<2;b++){const w=136,x=x0+80+a*140,z=z0+80+b*140,h=(70+hash(i*2+a,j*2+b)*300)*f;tower(x,z,w,h,pk(GLS));shopDoors(x,z,w,a?1:-1,b?1:-1)}};
 const offices=(i,j,x0,z0)=>{lot(x0,z0,'#4a4d55');lowWall(x0,z0,'#cfcac0',[],[]);const h=45+R(0,70),c=pk(GLS);
  add(bld,x0+150,h/2+.6,z0+90,200,h,110,C.set(c));add(roofs,x0+150,h+2,z0+90,190,3,100,C.set('#d5d8dc'));add(roofs,x0+120,h+8,z0+90,30,10,24,C.set('#bbbbbb'));
  carRow(x0+40,z0+180,7,38);carRow(x0+40,z0+250,7,38);palm(x0+20,z0+160,38);palm(x0+280,z0+160,34)};
 const mall=(i,j,x0,z0)=>{lot(x0,z0,'#4a4d55');lowWall(x0,z0,'#cfcac0',[],[]);const c=pk(STU),sc=pk(['#d62828','#1d4e89','#2a9d8f','#e9a820']);
  add(bld,x0+150,8.6,z0+34,258,16,32,C.set(c));add(roofs,x0+150,17.5,z0+34,262,2,36,C.set('#8d8a82'));add(roofs,x0+150,12,z0+50.4,250,5,1.5,C.set(sc));add(roofs,x0+150,9.5,z0+53,250,1.6,6,C.set(pk(['#e0524a','#2f7fc1','#3aa675'])));
  carRow(x0+30,z0+110,11,22);carRow(x0+30,z0+200,11,22);carRow(x0+30,z0+260,11,22);
  add(trunks,x0+262,22,z0+150,1.5,44,1.5,C.set('#555555'));add(roofs,x0+262,50,z0+150,3,16,22,C.set(sc))};
 const gas=(i,j,x0,z0)=>{lot(x0,z0,'#4a4d55');lowWall(x0,z0,'#cfcac0',[],[]);const sc=pk(['#d62828','#1d4e89','#f2b705']);
  add(roofs,x0+110,20,z0+150,90,3,50,C.set('#f2f2f2'));add(roofs,x0+110,21,z0+176.5,90,3.5,1.5,C.set(sc));
  for(const sx of[-36,36])for(const sz of[-16,16])add(roofs,x0+110+sx,10,z0+150+sz,2.5,20,2.5,C.set('#c8c8c8'));
  for(const sx of[-20,20])add(roofs,x0+110+sx,4,z0+150,4,7,4,C.set(sc));
  add(bld,x0+235,8.6,z0+75,40,16,48,C.set(pk(STU)));add(roofs,x0+235,17.5,z0+75,43,2,51,C.set('#8d8a82'));
  add(trunks,x0+60,24,z0+265,1.5,48,1.5,C.set('#555555'));add(roofs,x0+60,52,z0+265,14,16,2.5,C.set(sc))};
 const apts=(i,j,x0,z0)=>{lot(x0,z0,'#cfd6b8');lowWall(x0,z0,'#d8d2c4',[],[]);const c=pk(STU),H=26+R(0,8);
  add(bld,x0+150,H/2+.6,z0+50,256,H,56,C.set(c));add(bld,x0+45,H/2+.6,z0+170,50,H,176,C.set(c));add(bld,x0+255,H/2+.6,z0+170,50,H,176,C.set(c));
  for(const [x,z,w,d] of[[x0+150,z0+50,260,60],[x0+45,z0+170,54,180],[x0+255,z0+170,54,180]])add(roofs,x,H+2,z,w,3,d,C.set('#cfcac0'));
  add(roofs,x0+150,.85,z0+185,60,.4,40,C.set('#4fc3e8'));palm(x0+90,z0+130,40);palm(x0+210,z0+130,36);palm(x0+110,z0+240,34);palm(x0+190,z0+245,42)};
 const sig=(X,Z)=>{SIGSET.add(Math.round(X/CELL)+','+Math.round(Z/CELL));
  for(const [px,pz,sx,sz] of[[X-6,Z-6,1,1],[X+RW+6,Z+RW+6,-1,-1]]){add(lamp,px,12,pz,1,24,1,C.set('#3c4048'));
   add(roofs,px+sx*20,23,pz,40,1.2,1.2,C.set('#3c4048'));add(roofs,px,23,pz+sz*20,1.2,1.2,40,C.set('#3c4048'));
   [[px+sx*38,pz],[px,pz+sz*38]].forEach(([hx,hz])=>{add(roofs,hx,20,hz,3.5,9,3.5,C.set('#202226'));add(roofs,hx,23,hz,3.9,2,3.9,C.set('#3a2222'));add(roofs,hx,20,hz,3.9,2,3.9,C.set('#3a3a22'));add(roofs,hx,17,hz,3.9,2,3.9,C.set('#223a22'))})}};
 const street=(x0,z0)=>{add(lamp,x0+8,3,z0+130,2.2,6,2.2,C.set('#d62828'));add(roofs,x0+8,3,z0+170,4,6,4,C.set('#2f4f3a'));
  if(hash(x0,z0)<.35){add(roofs,x0+7,11.5,z0+250,6,.8,18,C.set('#5a6a7a'));add(roofs,x0+5,5.5,z0+242,.6,11,.6,C.set('#8a8f99'));add(roofs,x0+5,5.5,z0+258,.6,11,.6,C.set('#8a8f99'));add(roofs,x0+7,3,z0+250,3,1.2,14,C.set('#8a5a3a'));add(roofs,x0+4.5,7,z0+250,.5,8,12,C.set(pk(['#e63946','#ffd23f','#2a9d8f'])))}};
 function lots3(i,j,x0,z0){sig(x0-RW,z0-RW);street(x0,z0);const kd=blockKind(i,j);if(kd!='tower'&&kd!='mall'&&kd!='gas'&&kd!='apts'&&kd!='offices'&&kd!='resi'){V2B(i,j,x0,z0,kd);return}
  const dc=Math.hypot(i-N/2,j-N/2)/(N/2),f=clamp(1.5-dc,.45,1.5),tp=hash(i+3,j+9),sv=hash(i*7+1,j*5+2);
  lampAt(x0+6,z0+100);lampAt(x0+294,z0+200);lampAt(x0+100,z0+6);lampAt(x0+200,z0+294);
  for(const [px,pz] of[[x0+6,z0+40],[x0+6,z0+260],[x0+294,z0+60],[x0+294,z0+240],[x0+60,z0+6],[x0+240,z0+294]])palm(px,pz,R(34,48));
  if(tp<(dc<.35?.6:dc<.6?.25:.04))towers(i,j,x0,z0,f);
  else if(sv<.1)mall(i,j,x0,z0);else if(sv<.17)gas(i,j,x0,z0);else if(sv<.28)apts(i,j,x0,z0);else if(sv<.34)offices(i,j,x0,z0);else resi(i,j,x0,z0)}
 function lots2(i,j,x0,z0){
  const r=hash(i+3,j+9),A=x0+12,B=z0+12,cx=x0+150,cz=z0+150,f=clamp(1.5-Math.hypot(i-N/2,j-N/2)/(N/2),.45,1.5),pc=()=>PAL[Math.floor(R(0,7))];
  lampAt(x0+6,z0+100);lampAt(x0+294,z0+200);lampAt(x0+100,z0+6);lampAt(x0+200,z0+294);
  const unit=(x,z,w,h,c)=>{add(bld,x,h/2+.6,z,w,h,w,C.set(c));add(roofs,x,h+2,z,w-8,3,w-8,C.set(c).offsetHSL(0,0,.12));if(h>70)add(roofs,x+R(-w/4,w/4),h+8,z+R(-w/4,w/4),22,10,22,C.set('#b9b9b9'))};
  if(r<.22){const h=(150+hash(i,j)*170)*f,c=pc();
   add(roofs,cx,7.6,cz,276,14,276,C.set('#cfcac0'));
   add(bld,cx,h/2+14,cz,170,h,170,C.set(c));add(bld,cx,h+14+h*.15,cz,100,h*.3,100,C.set(c));
   add(roofs,cx,h*1.3+16,cz,110,3,110,C.set('#dddddd'));add(roofs,cx,h*1.3+36,cz,3,40,3,C.set('#cccccc'))}
  else if(r<.52){for(let a=0;a<2;a++)for(let b=0;b<2;b++){const w=136,x=x0+80+a*140,z=z0+80+b*140,h=(55+hash(i*2+a,j*2+b)*175)*f,c=pc();unit(x,z,w,h,c);
    const sx=a?1:-1,sz=b?1:-1,ac=['#e0524a','#2f7fc1','#e8a33d','#3aa675'][Math.floor(R(0,4))];
    add(roofs,x+sx*(w/2+3),9,z,7,2.5,w-8,C.set(ac));add(roofs,x,9,z+sz*(w/2+3),w-8,2.5,7,C.set(ac));shopDoors(x,z,w,sx,sz)}}
  else if(r<.8){add(bases,cx,.67,cz,276,.1,276,C.set('#a8dc9a'));hedge2(x0,z0,hash(i,j)<.5?'#4f9d5a':'#e9e6df',[A+46,A+138,A+230],[B+138]);
   for(let a=0;a<3;a++)for(let b=0;b<3;b++){const x=A+46+a*92,z=B+46+b*92,w=56,h=24+hash(i*3+a,j*3+b)*20;
    add(bld,x,h/2+.6,z,w,h,w,C.set(pc()));add(cone,x,h+10,z,w*.78,20,w*.78,C.set(['#c0564a','#5a6f9a','#7a5a48','#6a8f5a'][Math.floor(R(0,4))]));
    add(roofs,x+12,h+14,z-10,4,10,4,C.set('#8b6b55'));add(roofs,x,.8,z+w/2+9,22,.3,18,C.set('#bdbab2'));
    resDoor(x0,z0,a,b,x,z);if(R(0,1)<.6)tree(x+(a==2?-1:1)*34,z-26,'#5fb86f',7)}}
  else{add(bases,cx,.67,cz,276,.1,276,C.set('#8d9096'));hedge(x0,z0,'#9aa0a8');
   for(let b=0;b<2;b++){const z=B+45+b*186,h=55+hash(i,j+b)*40,c=pc();add(bld,cx,h/2+.6,z,250,h,66,C.set(c));add(roofs,cx,h+2,z,242,3,58,C.set(c).offsetHSL(0,0,.12))}
   for(let q=0;q<7;q++)add(roofs,A+25+q*37,3.6,cz,28,5.5,13,C.set(CARC[q%7]))}}
 function lots(i,j,x0,z0){
  const r=hash(i+3,j+9),cx=x0+150,cz=z0+150,f=clamp(1.5-Math.hypot(i-N/2,j-N/2)/(N/2),.45,1.5),pc=()=>PAL[Math.floor(R(0,7))];
  if(r<.3){const h=(130+hash(i,j)*160)*f,c=pc();
   add(bld,cx,h/2+1,cz,190,h,190,C.set(c));add(bld,cx,h+h*.15+1,cz,110,h*.3,110,C.set(c));
   add(roofs,cx,h*1.3+3,cz,120,3,120,C.set('#dddddd'));add(roofs,cx,h*1.3+22,cz,3,40,3,C.set('#cccccc'))}
  else if(r<.65){for(let a=0;a<2;a++)for(let b=0;b<2;b++){const h=(50+hash(i*2+a,j*2+b)*170)*f,w=112,x=x0+75+a*150,z=z0+75+b*150,c=pc();
    add(bld,x,h/2+1,z,w,h,w,C.set(c));add(roofs,x,h+2.5,z,w-14,3,w-14,C.set(c).offsetHSL(0,0,.12));
    if(h>110)add(roofs,x,h+9,z,24,12,24,C.set('#bbbbbb'))}}
  else{for(let a=0;a<3;a++)for(let b=0;b<3;b++){const w=62,h=24+hash(i*3+a,j*3+b)*22,x=x0+50+a*100,z=z0+50+b*100;
    add(bld,x,h/2+1,z,w,h,w,C.set(pc()));add(cone,x,h+10,z,w*.75,20,w*.75,C.set(['#c0564a','#5a6f9a','#7a5a48'][Math.floor(R(0,3))]))}}}
 const tree=(x,z,col,r)=>{add(trunks,x,r*.6,z,1.6,r*1.2,1.6,C.set('#8a6a4a'));add(trees,x,r*1.5,z,r,r,r,C.set(col))};
 for(let i=0;i<N;i++)for(let j=0;j<N;j++){const x0=i*CELL+RW,z0=j*CELL+RW,cx=x0+150,cz=z0+150;
  if(isWater(i,j)){rails2(i,j,x0,z0);continue}
  if(isPark(i,j)){add(bases,cx,.3,cz,300,.6,300,C.set('#9ad98f'));park2(i,j,x0,z0);
   for(let q=0;q<9;q++){const a=hash(i*9+q,j),b=hash(i,j*9+q);tree(x0+20+a*260,z0+20+b*260,q%3?'#4fae6a':'#f7a8c4',11+a*8)}continue}
  add(bases,cx,.3,cz,300,.6,300,C.set('#d9d6cf'));
  lots3(i,j,x0,z0);
  
  if(blockKind(i,j)=='tower'||blockKind(i,j)=='mall'||blockKind(i,j)=='gas'||blockKind(i,j)=='apts'||blockKind(i,j)=='offices'||blockKind(i,j)=='resi'||blockKind(i,j)=='alley'){for(const ox of[9,291])for(const oz of[9,291])tree(x0+ox,z0+oz,'#6cc08b',8)}}
}
/* ===== v2 world (early) ===== */
let BM=null,VW=null;
const ovSm=u=>{u=Math.max(0,Math.min(1,u));return u*u*(3-2*u)};
const FWJ=11,FWZ0=FWJ*CELL+RW,OVP=[3,7,13,17,25,29],OVH=28,OVZ0=FWJ*CELL-300,OVL=1100,RAMPX=[900,2500,4100,5700,7300,8900,10500,12100],GARB=[[14,14],[7,25],[24,8],[27,22]],XMIN=-110;
const TUN=[{h:1,j:8,i:6},{h:1,j:14,i:23},{h:0,i:13,j:21},{h:0,i:4,j:2}];
const bandF=z=>z>FWZ0&&z<FWZ0+300;
const KC=new Map();
function blockKind(i,j){const key=i+j*64,m=KC.get(key);if(m)return m;let k;
 if(isWater(i,j))k='water';else if(j==FWJ)k='fw';else if(i==0)k='beach';
 else if(GARB.some(g=>g[0]==i&&g[1]==j))k='garage';
 else if(j>=24&&i>=12&&i<=30){if(j==N-1)k='quay';else{const hh=hash(i*3+5,j*7+1);k=hh<.25?'walley':hh<.45?'ware':hh<.65?'yard':hh<.8?'lot':'ware'}}
 else if(hash(i,j)<.2)k='park';
 else{const dc=Math.hypot(i-N/2,j-N/2)/(N/2),tp=hash(i+3,j+9),sv=hash(i*7+1,j*5+2);
  if(tp<(dc<.35?.6:dc<.6?.25:.04))k=hash(i*13+2,j*3+7)<.4?'alley':'tower';
  else k=sv<.1?'mall':sv<.17?'gas':sv<.28?'apts':sv<.34?'offices':'resi'}
 KC.set(key,k);return k}
const OPENK={park:1,beach:1,fw:1,lot:1,quay:1,garage:1},NOCOL={park:1,beach:1,fw:1,lot:1,quay:1,garage:1,alley:1,walley:1},ALLEYK={alley:1,walley:1};
const isOpen=(i,j)=>!!OPENK[blockKind(i,j)];
const BCOL={water:'#3d7fb0',fw:'#7a7f8a',beach:'#d9c98a',garage:'#5a6a8a',lot:'#6a6a70',quay:'#8a8a8a',ware:'#5d5a52',walley:'#5d5a52',yard:'#7a5a40',park:'#3f7a4c',alley:'#46505f'};
function blockColor(i,j){return BCOL[blockKind(i,j)]||'#46505f'}
function ovH(x,z){if(z<OVZ0||z>OVZ0+OVL)return 0;for(const i of OVP)if(x>i*CELL-10&&x<i*CELL+RW+10){const t=(z-OVZ0)/OVL;return OVH*ovSm(Math.min(t,1-t)/.25)}return 0}
const COVERS=[],NOPED=[[0,FWZ0-12,SIZE,FWZ0+312],[-3000,-3000,-5,SIZE+3000]],FWB=[],PILL=new Map();
(function(){const seg=(y0,y1,gw)=>{let s=0;for(const rx of RAMPX){FWB.push([s,y0,rx-gw,y1]);s=rx+gw}FWB.push([s,y0,SIZE,y1])};seg(FWZ0,FWZ0+8,75);seg(FWZ0+292,FWZ0+300,75);seg(FWZ0+143,FWZ0+157,45);
 for(const q of TUN){const r=q.h?[q.i*CELL+RW,q.j*CELL,(q.i+1)*CELL,q.j*CELL+RW]:[q.i*CELL,q.j*CELL+RW,q.i*CELL+RW,(q.j+1)*CELL];COVERS.push(r);NOPED.push(r)}})();
const inRect=(L,x,y)=>L.some(q=>x>q[0]&&x<q[2]&&y>q[1]&&y<q[3]),inCover=(x,y)=>inRect(COVERS,x,y);
const VB=inst(gBox,mat('#fff')),VC=inst(gCyl,mat('#fff')),VS=inst(gSph,mat('#fff'));
function vb(x,y,z,sx,sy,sz,c,q){add(VB,x,y,z,sx,sy,sz,C.set(c),q)}
function vc(x,y,z,sx,sy,sz,c){add(VC,x,y,z,sx,sy,sz,C.set(c))}
function vs(x,y,z,sx,sy,sz,c){add(VS,x,y,z,sx,sy,sz,C.set(c))}
function vpalm(x,z,h){vc(x,h/2,z,1.4,h,1.4,'#9a8460');vs(x,h+1,z,9,3,9,'#4f9a4a');vs(x,h+3.5,z,5.5,2.4,5.5,'#63b05a')}
function addPill(i,j,x,z){const k=i+','+j;let a=PILL.get(k);if(!a){a=[];PILL.set(k,a)}a.push([x-5,z-5,x+5,z+5])}
const IC=['#c0392b','#2c6fbb','#e0a030','#2f8f5f','#8a8f99','#d35400','#7f5a9a'],WALLC=['#b8c4cc','#c9c2b0','#9fb0bc','#d6d0c2'],GLS2=['#6f8fa8','#8aa6b8','#5f7c92','#a8b8c4','#7d93a5','#4e6a80'];
const pk2=a=>a[Math.floor(Math.random()*a.length)];
function V2B(i,j,x0,z0,kd){
 const A=x0+12,B=z0+12,cx=x0+150,cz=z0+150;
 if(kd=='fw')return;
 if(kd=='beach'){vb(cx,.67,cz,300,.1,300,'#ead9a6');vb(x0+12,.85,cz,10,.3,300,'#b08a5a');
  for(let q=0;q<7;q++)vpalm(x0+26,z0+20+q*42,R(34,46));
  for(let q=0;q<9;q++){const x=x0+60+R(0,220),z=z0+20+R(0,260),c=pk2(['#e63946','#ffd23f','#2a9d8f','#f4f4f4','#4fc3e8']);vc(x,7,z,.7,14,.7,'#dddddd');vs(x,14,z,9,2.4,9,c);vb(x+14,.85,z+4,8,.2,14,pk2(['#e63946','#ffd23f','#4fc3e8','#f4a6b7']))}
  vb(x0+130,6,z0+60,3,12,3,'#cccccc');vb(x0+190,6,z0+60,3,12,3,'#cccccc');vb(x0+160,9,z0+60,60,7,.6,'#f4f4f4');
  vb(x0+210,10,z0+230,22,2,22,'#f4f4f4');for(const sx of[-9,9])for(const sz of[-9,9])vb(x0+210+sx,5,z0+230+sz,2,10,2,'#e63946');vb(x0+210,15,z0+230,16,8,16,'#e63946');vb(x0+210,20,z0+230,20,1.5,20,'#f4f4f4');return}
 if(kd=='garage'){vb(cx,.67,cz,300,.1,300,'#3a3d44');
  for(let a=0;a<4;a++)for(let b=0;b<4;b++){const px=x0+50+a*66,pz=z0+50+b*66;vb(px,13,pz,9,26,9,'#9a9a9a');addPill(i,j,px,pz)}
  for(let q=0;q<9;q++)vb(x0+24+q*33,.8,cz,.8,.2,250,'#f4f4f4');
  const r=new THREE.Mesh(new THREE.BoxGeometry(300,2,300),new THREE.MeshLambertMaterial({color:0x6b6f78,transparent:true,opacity:.38}));r.position.set(cx,27,cz);scene.add(r);
  vb(cx,36,z0+6,46,16,2,'#1d4e89');vb(cx,36,z0+4.8,34,10,.6,'#ffffff');COVERS.push([x0,z0,x0+300,z0+300]);return}
 if(kd=='lot'){vb(cx,.67,cz,300,.1,300,'#5a5d66');for(let q=0;q<10;q++)vb(x0+25+q*28,.8,cz,.8,.2,260,'#f2c230');for(let q=0;q<14;q++)vc(x0+20+R(0,260),2,z0+20+R(0,260),2,4,2,'#ff7a1a');return}
 if(kd=='quay'){vb(cx,.67,cz,300,.1,300,'#9a9a9a');for(let q=0;q<8;q++)vc(x0+20+q*37,3,z0+284,2.5,6,2.5,'#444444');
  if(i%3==0){const x=x0+150;for(const dx of[-40,40]){vb(x+dx,40,z0+296,5,80,5,'#e0a030');vb(x+dx,40,z0+330,5,80,5,'#e0a030')}vb(x,82,z0+313,100,5,40,'#e0a030');vb(x,76,z0+330,6,4,70,'#c9a227')}
  for(let q=0;q<4;q++)vb(x0+30+q*70,5,z0+60+R(0,100),48,10,18,pk2(IC));return}
 if(kd=='ware'){const f=hash(i,j)<.35;
  if(f){vb(cx,24,z0+215,276,48,140,pk2(WALLC));vb(cx,48.5,z0+215,280,2,144,'#6b6f78');for(let q=0;q<3;q++){vc(A+50+q*80,22,z0+70,38,44,38,'#c9cfd4');vc(A+50+q*80,45,z0+70,38,2,38,'#8a8f99')}
   vc(x0+60,45,z0+200,5,90,5,'#8a8a8a');vc(x0+110,40,z0+200,4.5,80,4.5,'#8a8a8a')}
  else for(let b=0;b<2;b++){const z=B+67+b*142,h=30+R(0,22),c=pk2(WALLC);vb(cx,h/2+.6,z,276,h,134,c);vb(cx,h+1.5,z,280,2,138,'#6b6f78');
   for(let q=0;q<6;q++)vb(A+25+q*45,6,b?z-67.5:z+67.5,26,12,.8,'#3a3f48');vb(cx,h+4,z,30,6,20,'#9a9a9a')}
  return}
 if(kd=='yard'){vb(cx,.67,cz,300,.1,300,'#6e6e72');
  for(let c=0;c<4;c++)for(let r=0;r<8;r++){const n=1+Math.floor(R(0,3)),x=A+36+c*68,z=A+17+r*34;for(let s=0;s<n;s++)vb(x,4.7+s*9.2,z,62,9,30,pk2(IC))}
  vb(x0+40,50,z0+150,6,100,6,'#e0a030');vb(x0+260,50,z0+150,6,100,6,'#e0a030');vb(cx,100,z0+150,226,5,8,'#e0a030');return}
 if(kd=='walley'){for(const a of[0,1]){const x=A+59+a*158,h=34+R(0,18),c=pk2(WALLC);vb(x,h/2+.6,cz,118,h,276,c);vb(x,h+1.5,cz,122,2,280,'#6b6f78');for(let q=0;q<5;q++)vb(a?x-59:x+59,6,A+30+q*55,.8,12,26,'#3a3f48')}
  vb(A+118+12,4,z0+60,6,8,10,'#2f6a3a');vb(A+158-12,4,z0+230,6,8,10,'#2f6a3a');COVERS.push([x0+130,z0+12,x0+170,z0+288]);return}
 if(kd=='alley'){if(!VW)VW=inst(gBox,BM);
  for(const a of[0,1])for(const b of[0,1]){const x=A+59+a*158,z=B+69+b*138,h=(90+hash(i*2+a,j*2+b)*260)*clamp(1.5-Math.hypot(i-N/2,j-N/2)/(N/2),.5,1.5),c=pk2(GLS2);
   add(VW,x,h/2+.6,z,118,h,138,C.set(c));vb(x,h+2,z,100,3,120,'#d5d8dc');if(h>200)vb(x,h+22,z,2.5,40,2.5,'#c8ccd0')}
  for(let q=0;q<4;q++){vb(A+118+4,3,B+30+q*65,6,6,10,'#2f6a3a');vb(A+158-4,12,B+50+q*65,1.2,20,8,'#6a6a70')}
  COVERS.push([x0+130,z0+12,x0+170,z0+288]);return}
}
function buildWorld2(){
 const plane=(w,h,x,y,z,c,tr)=>{const m=new THREE.Mesh(new THREE.PlaneGeometry(w,h),tr?new THREE.MeshLambertMaterial({color:c,transparent:true,opacity:tr,depthWrite:false}):mat(c));m.rotation.x=-PI/2;m.position.set(x,y,z);scene.add(m);return m};
 plane(3000,SIZE+6000,-140-1500,.12,SIZE/2,'#3d9bd0');plane(140,SIZE,-70,.14,SIZE/2,'#ead9a6');plane(SIZE+6000,3000,SIZE/2,.12,SIZE+1500,'#3d9bd0');
 // pier on the beach
 vb(-420,1.4,CELL*16+60,560,2,24,'#9a7a52');for(let q=0;q<14;q++){vc(-160-q*38,-2,CELL*16+60-10,2,10,2,'#6a5238');vc(-160-q*38,-2,CELL*16+60+10,2,10,2,'#6a5238')}
 for(let q=0;q<3;q++)vb(SIZE*.4+q*900,12,SIZE+420+q*120,160,24,40,pk2(['#c0392b','#2c6fbb','#5a6a7a']));
 // freeway
 plane(SIZE,300,SIZE/2,.8,FWZ0+150,'#4b4f58');
 for(let x=0;x<SIZE;x+=44)for(const dz of[55,93,207,245])vb(x+12,.95,FWZ0+dz,24,.2,2,'#f2f2f2');
 for(let x=0;x<SIZE;x+=400)for(const dz of[17,131,169,283])vb(x+200,.95,FWZ0+dz,400,.2,2.2,(dz==131||dz==169)?'#ffd23f':'#f2f2f2');
 const inGap=(x,gw)=>RAMPX.some(rx=>Math.abs(x-rx)<gw);
 for(let x=20;x<SIZE;x+=40){if(!inGap(x,45))vb(x,3.2,FWZ0+150,40,6.4,12,'#c9c6bd');
  if(!inGap(x,75)){vb(x,2.6,FWZ0+4,40,5.2,6,'#b9b6ad');vb(x,2.6,FWZ0+296,40,5.2,6,'#b9b6ad')}}
 for(const rx of RAMPX)for(let q=0;q<5;q++){vb(rx-60+q*30,.95,FWZ0-6,16,.2,3,'#ffd23f');vb(rx-60+q*30,.95,FWZ0+306,16,.2,3,'#ffd23f')}
 for(let x=60;x<SIZE;x+=200){vc(x,15,FWZ0+150,.9,30,.9,'#5a5f68');vb(x,30,FWZ0+135,1.2,1.2,32,'#5a5f68');vb(x,30,FWZ0+165,1.2,1.2,32,'#5a5f68');vb(x,29,FWZ0+120,4,1.4,5,'#fff3b0');vb(x,29,FWZ0+180,4,1.4,5,'#fff3b0')}
 const sgn=(txt,sub,x,z,col)=>{const cv=document.createElement('canvas');cv.width=512;cv.height=160;const c2=cv.getContext('2d');c2.fillStyle=col;c2.fillRect(0,0,512,160);c2.strokeStyle='#fff';c2.lineWidth=8;c2.strokeRect(6,6,500,148);c2.fillStyle='#fff';c2.font='bold 64px sans-serif';c2.textAlign='center';c2.fillText(txt,256,80);c2.font='bold 40px sans-serif';c2.fillText(sub,256,134);
  const m=new THREE.Mesh(new THREE.PlaneGeometry(88,27.5),new THREE.MeshBasicMaterial({map:new THREE.CanvasTexture(cv),side:THREE.DoubleSide}));m.rotation.y=PI/2;m.position.set(x,44,z);scene.add(m)};
 [[1500,'EXIT 12','Downtown'],[3700,'EXIT 14','Beach 6 mi'],[6300,'I-5 NORTH','Hollywood Mtn'],[9100,'EXIT 18','Port / Industrial'],[11500,'EXIT 20','Mountain Rd']].forEach(([x,a,b])=>{
  vc(x,18,FWZ0+4,2.4,36,2.4,'#7a7f88');vc(x,18,FWZ0+296,2.4,36,2.4,'#7a7f88');vb(x,36,FWZ0+150,3,3,300,'#7a7f88');sgn(a,b,x,FWZ0+85,'#1b7a43');sgn(a,b,x,FWZ0+215,'#1b7a43')});
 for(const i of OVP){const x=i*CELL+50,Wd=RW+20,n=44,L=OVL;
  const prof=(lo,hi)=>{const top=[],bot=[];for(let k=0;k<=n;k++){const t=k/n,hh=OVH*ovSm(Math.min(t,1-t)/.25);top.push([t*L,.9+hh+hi]);bot.push([t*L,.9+hh+lo])}return top.concat(bot.reverse())};
  const strip=(lo,hi,w,off,m)=>{const g=ext(prof(lo,hi),w);g.translate(0,0,off);const o=new THREE.Mesh(g,m);o.rotation.y=-PI/2;o.position.set(x,0,OVZ0);scene.add(o)};
  strip(-6,0,Wd,0,mat('#4a505c'));strip(.01,.08,3,0,mat('#f4e9a8'));strip(-6,5,3,Wd/2-1.5,mat('#cfcac0'));strip(-6,5,3,-(Wd/2-1.5),mat('#cfcac0'));
  for(const dx of[-35,35])vb(x+dx,11.5,FWZ0+150,8,23,8,'#a8a398')}
 // tunnels
 for(const q of TUN){const r=q.h?[q.i*CELL+RW,q.j*CELL,(q.i+1)*CELL,q.j*CELL+RW]:[q.i*CELL,q.j*CELL+RW,q.i*CELL+RW,(q.j+1)*CELL],w=r[2]-r[0],d=r[3]-r[1],cx=(r[0]+r[2])/2,cz=(r[1]+r[3])/2;
  const roof=new THREE.Mesh(new THREE.BoxGeometry(w+(q.h?0:12),3,d+(q.h?12:0)),new THREE.MeshLambertMaterial({color:0x4a4f58,transparent:true,opacity:.42}));roof.position.set(cx,28,cz);scene.add(roof);
  if(q.h){vb(cx,14,r[1]-3,w,28,4,'#8d8a82');vb(cx,14,r[3]+3,w,28,4,'#8d8a82');for(let x=r[0]+20;x<r[2];x+=40)vb(x,26,cz,4,1,40,'#ffe9a0');
   for(const x of[r[0]+1,r[2]-1]){vb(x,31,cz,4,6,d+12,'#6a6f78')}}
  else{vb(r[0]-3,14,cz,4,28,d,'#8d8a82');vb(r[2]+3,14,cz,4,28,d,'#8d8a82');for(let z=r[1]+20;z<r[3];z+=40)vb(cx,26,z,40,1,4,'#ffe9a0');
   for(const z of[r[1]+1,r[3]-1]){vb(cx,31,z,d+12,6,4,'#6a6f78')}}}
}

build();
buildMountain();buildWorld2();finInst();
/* particles */
const MAXP=500,pg=new THREE.BufferGeometry(),pp=new Float32Array(MAXP*3),pcl=new Float32Array(MAXP*3);
pg.setAttribute('position',new THREE.BufferAttribute(pp,3));pg.setAttribute('color',new THREE.BufferAttribute(pcl,3));
const pts=new THREE.Points(pg,new THREE.PointsMaterial({size:6,vertexColors:true,depthWrite:false}));pts.frustumCulled=false;scene.add(pts);
const skid=new THREE.InstancedMesh(gBox,new THREE.MeshBasicMaterial({color:0x222222}),600),SQ=new THREE.Quaternion(),SE=new THREE.Euler();skid.frustumCulled=false;skid.count=0;scene.add(skid);let skN=0;
function addSkid(x,z,a){SE.set(0,-a,0);SQ.setFromEuler(SE);M4.compose(P.set(x,.7+gh(x,z),z),SQ,S.set(8,.1,2.4));skid.setMatrixAt(skN%600,M4);skN++;skid.count=Math.min(skN,600);skid.instanceMatrix.needsUpdate=true}
/* ---------- game logic ---------- */
let p,cars,peds,parts,pops,t,score,combo,comboT,shake,lvl,spawnCd,inv,coins;
function mkCar(x,y,a,type,col,kind,opt){const c={x,y,a,vx:0,vy:0,t:type,col,hp:100,acc:.085,free:false,wreck:false,life:0,d:0,sp:2.5,li:null,stuck:0,rev:0,hitCd:0};
 c.mesh=carMesh2(type,col,kind,opt);scene.add(c.mesh);return c}
const laneOff=d=>d==0?-14:d==1?14:d==2?14:-14;
function spawnPt(a0,a1){const a=R(0,6.283),d=R(a0,a1);let x=clamp(p.x+Math.cos(a)*d,150,SIZE-150),y=clamp(p.y+Math.sin(a)*d,150,SIZE-150),h;
 const k=v=>clamp(Math.round((v-50)/CELL),1,N-2)*CELL+50;
 if(Math.random()<.5){y=k(y);h=true}else{x=k(x);h=false}if(h?hClosed(x,Math.round((y-50)/CELL)):vClosed(y,Math.round((x-50)/CELL)))return spawnPt(a0,a1);return{x,y,h}}
function start(){
 if(cars){cars.forEach(kill);peds.forEach(kill);coins.forEach(kill);kill(p)}
 p=mkCar(12*CELL+50,12*CELL+50,0,'p','#ff5a5f');p.acc=.0616;
 cars=[];peds=[];coins=[];parts=[];pops=[];t=0;score=0;combo=0;comboT=0;shake=0;lvl=1;spawnCd=0;inv=0;stars=1;heat=25;money=0;bustT=0;notes=[];flash=0;spawnT=0;recT=0;POL={state:'chase',lx:p.x,ly:p.y,lvx:0,lvy:0,lostT:0,searchT:0,seen:false,time:0,maxS:1};endMsg='BUSTED!';state='play';
 for(let i=0;i<20;i++)spawnTraffic(150,1100);
 for(let i=0;i<60;i++)spawnPed(100,1200);
 spawnCop(450,650);
 for(let i=0;i<70;i++)spawnCoin(100,1300);
 addSpiralCoins();
}
function spawnTraffic(a0=650,a1=1050){const s=spawnPt(a0,a1);const d=s.h?(Math.random()<.5?0:2):(Math.random()<.5?1:3);
 if(s.h)s.y+=laneOff(d);else s.x+=laneOff(d);
 if(Math.hypot(s.x-p.x,s.y-p.y)<120)return;
 const c=mkCar(s.x,s.y,d*PI/2,'t',CARC[Math.floor(R(0,CARC.length))]);c.d=d;c.sp=R(1.4,2.25);cars.push(c)}
function spawnPolice(a0=700,a1=1000){const s=spawnPt(a0,a1);const c=mkCar(s.x,s.y,Math.atan2(p.y-s.y,p.x-s.x),'police','#fff');
 c.acc=.7*(.062+.005*lvl);cars.push(c)}
function spawnPed(a0=600,a1=1100){const s=spawnPt(a0,a1);const o=(Math.random()<.5?-1:1)*(RW/2+6);
 if(s.h)s.y+=o;else s.x+=o;if(!onRoad(s.x,s.y))return;const col=PAL[Math.floor(R(0,7))];peds.push({x:s.x,y:s.y,vx:0,vy:0,tm:0,hit:0,col,ph:R(0,6),fa:R(0,6.28),mode:0,ws:R(.45,.85),ins:0,door:null,tvx:0,tvy:0,tx:0,ty:0,mesh:pedMesh4(col)});scene.add(peds[peds.length-1].mesh)}
const onRoad=(x,y,m=10)=>!inCL(x,y)&&!inRect(NOPED,x,y)&&(((x+m)%CELL<RW+2*m)||((y+m)%CELL<RW+2*m));
function stepCar(c,thr,st,hb,k){
 const ca=Math.cos(c.a),sa=Math.sin(c.a);let f=c.vx*ca+c.vy*sa,l=-c.vx*sa+c.vy*ca;
 f+=thr*c.acc*k;f*=Math.pow(.991,k);if(thr==0&&!hb)f*=Math.pow(.993,k);l*=Math.pow(hb?.97:.86,k);if(hb)f*=Math.pow(.985,k);
 c.a+=st*.055*clamp(f/3,-1,1)*(1-.42*clamp(Math.abs(f)/7,0,1))*k;
 const c2=Math.cos(c.a),s2=Math.sin(c.a);c.vx=f*c2-l*s2;c.vy=f*s2+l*c2;c.x+=c.vx*k;c.y+=c.vy*k}
function collideBlocks(c,r){let imp=collideBlocks0(c,r);imp=Math.max(imp,mRails(c,r));
 for(const q of CL){if(Math.abs(c.x-(q[0]+q[2])/2)>(q[2]-q[0])/2+r+2||Math.abs(c.y-(q[1]+q[3])/2)>(q[3]-q[1])/2+r+2)continue;
  const qx=clamp(c.x,q[0],q[2]),qy=clamp(c.y,q[1],q[3]);let dx=c.x-qx,dy=c.y-qy,d=Math.hypot(dx,dy);
  if(d<r){if(d<.001){dx=0;dy=-1;d=1}const nx=dx/d,ny=dy/d;c.x=qx+nx*r;c.y=qy+ny*r;const vn=c.vx*nx+c.vy*ny;if(vn<0){c.vx-=1.5*vn*nx;c.vy-=1.5*vn*ny;imp=Math.max(imp,-vn)}}}
 return imp}
function collideBlocks0(c,r){let imp=0;const ci=Math.floor(c.x/CELL),cj=Math.floor(c.y/CELL);
 for(let i=ci-1;i<=ci+1;i++)for(let j=cj-1;j<=cj+1;j++){if(i<0||j<0||i>=N||j>=N||isPark(i,j))continue;
  const x0=i*CELL+RW+12,y0=j*CELL+RW+12,qx=clamp(c.x,x0,x0+276),qy=clamp(c.y,y0,y0+276);
  let dx=c.x-qx,dy=c.y-qy,d=Math.hypot(dx,dy);
  if(d<r){if(d<.001){dx=0;dy=-1;d=1}const nx=dx/d,ny=dy/d;c.x=qx+nx*r;c.y=qy+ny*r;const vn=c.vx*nx+c.vy*ny;
   if(vn<0){c.vx-=1.5*vn*nx;c.vy-=1.5*vn*ny;imp=Math.max(imp,-vn)}}}
 if(c.x<60){c.x=60;c.vx=Math.abs(c.vx)*.5}if(c.x>XMAX-60){c.x=XMAX-60;c.vx=-Math.abs(c.vx)*.5}
 if(c.y<60){c.y=60;c.vy=Math.abs(c.vy)*.5}if(c.y>SIZE-60){c.y=SIZE-60;c.vy=-Math.abs(c.vy)*.5}
 return imp}
function driveT(c,k){
 const h=c.d%2==0,ang=c.d*PI/2;let da=((ang-c.a+PI*3)%(PI*2))-PI;c.a+=da*.12*k;
 if(c.cs===undefined)c.cs=c.sp;c.cs+=(trafficSpeed(c)-c.cs)*.12*k;c.vx=Math.cos(c.a)*c.cs;c.vy=Math.sin(c.a)*c.cs;
 if(h){c.y+=((Math.round((c.y-50)/CELL)*CELL+50+laneOff(c.d))-c.y)*.08*k}else{c.x+=((Math.round((c.x-50)/CELL)*CELL+50+laneOff(c.d))-c.x)*.08*k}
 c.x+=c.vx*k;c.y+=c.vy*k;
 const al=h?c.x:c.y,ic=Math.round((al-50)/CELL)*CELL+50;
 if(Math.abs(al-ic)<c.sp*k&&c.li!==ic){c.li=ic;
  const ix=Math.round((c.x-50)/CELL)*CELL+50,iy=Math.round((c.y-50)/CELL)*CELL+50,ops=[];
  const D=[[1,0],[0,1],[-1,0],[0,-1]];
  for(let d=0;d<4;d++){if(d==(c.d+2)%4)continue;const nx=ix+D[d][0]*CELL,ny=iy+D[d][1]*CELL;
   if(nx>=450&&nx<=50+(N-2)*CELL&&ny>=450&&ny<=50+(N-2)*CELL&&!segClosed(ix,iy,nx,ny)){const w=d==c.d?3:1;for(let q=0;q<w;q++)ops.push(d)}}
  c.d=ops.length?ops[Math.floor(Math.random()*ops.length)]:(c.d+2)%4}}
function policeAI(c,k){
 const T=policeGoal(c,k);let da=Math.atan2(T.y-c.y,T.x-c.x)-c.a;da=((da+PI*3)%(PI*2))-PI;
 const sp=Math.hypot(c.vx,c.vy);let thr=c.hold>0?(c.hold-=k,0):1,st=clamp(da*2,-1,1);if(Math.abs(da)>.9&&sp>3.2&&thr>0)thr=.2;
 if(sp<.6)c.stuck+=k;else c.stuck=0;
 if(c.stuck>50){c.rev=40;c.stuck=0}
 if(c.rev>0){c.rev-=k;thr=-.8;st=-st}
 stepCar(c,thr,st,false,k)}
function pop(x,y,txt,col){pops.push({x,y,txt,col,l:60})}
function burst(x,y,col,n=8){for(let i=0;i<n&&parts.length<MAXP;i++)parts.push({x,y,h:R(3,10),vx:R(-3,3),vy:R(-3,3),vh:R(0,2.5),g:.1,l:R(20,40),c:new THREE.Color(col)})}
function smoke(x,y){if(parts.length<MAXP)parts.push({x,y,h:12,vx:R(-.5,.5),vy:R(-.5,.5),vh:.4,g:0,l:45,c:new THREE.Color('#777')})}
function hurt(d){if(inv>0||d<=0)return;p.hp-=d;inv=8;shake=Math.max(shake,Math.min(10,d*.5))}
function carHit(a,b){
 const dx=b.x-a.x,dy=b.y-a.y,d=Math.hypot(dx,dy);if(d>=30||d<.01)return;
 if(a.t=='blk'||b.t=='blk'){const w=a.t=='blk'?a:b,o=w===a?b:a;if(o.t=='blk')return;const ddx=o.x-w.x,ddy=o.y-w.y,dd=Math.hypot(ddx,ddy)||1,nx2=ddx/dd,ny2=ddy/dd;o.x=w.x+nx2*30;o.y=w.y+ny2*30;const vn=o.vx*nx2+o.vy*ny2;
  if(vn<0){o.vx-=1.4*vn*nx2;o.vy-=1.4*vn*ny2;const im=-vn;if(o.t=='p'){if(im>1.2){hurt((im-1.2)*3);burst(o.x,o.y,'#fff',5)}}else if(o.t=='t'&&im>1.5&&!o.free){o.free=true;tintCar(o,'#999')}}return}
 if(Math.abs(gh(a.x,a.y)-gh(b.x,b.y))>12)return;
 if(a.t=='t'&&!a.free&&b.t=='t'&&!b.free)return;
 const nx=dx/d,ny=dy/d,ov=(30-d)/2;a.x-=nx*ov;a.y-=ny*ov;b.x+=nx*ov;b.y+=ny*ov;
 const rn=(b.vx-a.vx)*nx+(b.vy-a.vy)*ny;if(rn>=0)return;const j=Math.min(-1.3*rn/2,2.6);
 a.vx-=j*nx;a.vy-=j*ny;b.vx+=j*nx;b.vy+=j*ny;const imp=-rn;if(imp<1)return;
 burst((a.x+b.x)/2,(a.y+b.y)/2,'#ffd86b',5);
 for(const [c,o] of [[a,b],[b,a]]){
  if(c.t=='p'){hurt((imp-1.5)*(o.t=='police'?3:2))}
  else if(c.t=='t'){if(imp>1.5&&!c.free){c.free=true;tintCar(c,'#999')}
   if(o.t=='p'&&c.hitCd<=0&&imp>3){c.hitCd=30;crime(8);combo++;comboT=120;const s=200*Math.min(combo,6);score+=s;pop(c.x,c.y,'クラッシュ! +'+s,'#ffd86b')}}
  else if(c.t=='police'&&!c.wreck){c.hp-=imp*(o.t=='p'?3.6:2);if(o.t=='p')crime(imp*.8);
   if(c.hp<=0){c.wreck=true;c.life=300;tintCar(c,'#555');c.mesh.l1.visible=c.mesh.l2.visible=false;if(o.t=='p'){score+=500;crime(20);combo++;comboT=120;pop(c.x,c.y,'パトカー撃破! +500','#ff7a8a')}}}}}
function update(dt){
 const k=Math.min(dt*60,2.5);t+=dt;lvl=stars;if(K.KeyR){K.KeyR=0;recoverPlayer()}polUpdate(dt,k);if(inv>0)inv-=k;if(shake>0)shake*=.9;
 comboT-=k;if(comboT<=0)combo=0;
 let thr=0,st=(K.KeyD?1:0)-(K.KeyA?1:0);if(K.KeyW)thr=1;
 const f=p.vx*Math.cos(p.a)+p.vy*Math.sin(p.a);
 const drift=(K.KeyS&&st!=0&&f>2.5)||!!K.Space;
 if(K.KeyS&&!drift){if(f>.8){p.vx*=Math.pow(.94,k);p.vy*=Math.pow(.94,k)}else thr=-.6}
 p.acc=.0616*(.55+.45*clamp(p.hp/60,0,1));stepCar(p,thr,drift?st*1.5:st,drift,k);
 if(drift&&Math.hypot(p.vx,p.vy)>2){for(const s of[-8,8])addSkid(p.x-Math.cos(p.a)*10-Math.sin(p.a)*s,p.y-Math.sin(p.a)*10+Math.cos(p.a)*s,p.a);smoke(p.x-Math.cos(p.a)*14,p.y-Math.sin(p.a)*14)}
 const ip=collideBlocks(p,15);if(ip>2.5){hurt((ip-2)*2.4);burst(p.x,p.y,'#fff',6)}
 if(p.hp<35&&Math.random()<.3)smoke(p.x,p.y);
 for(const c of cars){
  if(c.hitCd>0)c.hitCd-=k;
  if(c.t=='police'){if(c.wreck){c.life-=k;stepCar(c,0,0,true,k);collideBlocks(c,15);if(Math.random()<.4)smoke(c.x,c.y)}
   else{policeAI(c,k);collideBlocks(c,15)}}
  else if(c.free){c.life+=k;stepCar(c,0,0,true,k);collideBlocks(c,15)}
  else driveT(c,k)}
 const all=[p,...cars];
 for(let i=0;i<all.length;i++)for(let j=i+1;j<all.length;j++){const a=all[i],b=all[j];if(Math.abs(a.x-b.x)<32&&Math.abs(a.y-b.y)<32)carHit(a,b)}
 const sp=Math.hypot(p.vx,p.vy);
 for(let i=peds.length-1;i>=0;i--){const q=peds[i];
  if(q.hit>0){q.x+=q.vx*k;q.y+=q.vy*k;q.vx*=.94;q.vy*=.94;q.hit-=k;if(q.hit<=0){kill(q);peds.splice(i,1)}continue}
  const dx=q.x-p.x,dy=q.y-p.y,d=Math.hypot(dx,dy);
  if(q.mode!=3&&d<19&&sp>1.5){q.hit=45;addCoin(q.x+R(-10,10),q.y+R(-10,10),50);crime(12);q.vx=p.vx*1.3;q.vy=p.vy*1.3;combo++;comboT=120;const s=100*Math.min(combo,8);score+=s;pop(q.x,q.y,'+'+s,'#fff');burst(q.x,q.y,q.col,6);continue}
  pedAI(q,k,dx,dy,d,sp);continue;
  if(d<140&&sp>1){vx=dx/d*2.2;vy=dy/d*2.2}else if(q.tm<=0){q.tm=R(40,140);const a=Math.floor(R(0,4))*PI/2;vx=Math.cos(a)*.6;vy=Math.sin(a)*.6}
  q.vx=vx;q.vy=vy;const nx=q.x+vx*k,ny=q.y+vy*k;if(onRoad(nx,ny)){q.x=nx;q.y=ny}else{q.vx=-vx;q.vy=-vy;q.tm=0}}
 for(let i=cars.length-1;i>=0;i--){const c=cars[i],d=Math.hypot(c.x-p.x,c.y-p.y);
  if(d>1800||(c.t=='t'&&c.free&&c.life>600)||(c.t=='police'&&c.wreck&&c.life<=0)){kill(c);cars.splice(i,1)}}
 for(let i=coins.length-1;i>=0;i--){const c=coins[i],d=Math.hypot(c.x-p.x,c.y-p.y);
  if(d<24){score+=c.val;pop(c.x,c.y,'💰 +'+c.val,'#ffd84a');burst(c.x,c.y,'#ffd84a',8);kill(c);coins.splice(i,1);continue}
  if(d>1700&&!c.keep){kill(c);coins.splice(i,1)}}
 while(coins.length<150)spawnCoin();
 while(peds.length<60)spawnPed();
 let tc=0,pc=0;for(const c of cars){if(c.t=='t')tc++;else if(!c.wreck)pc++}
 if(tc<20)spawnTraffic();spawnCd-=k;
 
 for(let i=parts.length-1;i>=0;i--){const q=parts[i];q.x+=q.vx*k;q.y+=q.vy*k;q.h=Math.max(0,q.h+q.vh*k);q.vh-=q.g*k;q.l-=k;if(q.l<=0)parts.splice(i,1)}
 for(let i=pops.length-1;i>=0;i--){pops[i].l-=k;if(pops[i].l<=0)pops.splice(i,1)}
 if(p.hp<=0){p.hp=0;endGame('WRECKED!')}
}
/* ---------- render ---------- */
const V=new THREE.Vector3();let camA=0;
function sync(c){c.mesh.position.set(c.x,.4+gh(c.x,c.y),c.y);c.mesh.rotation.order='YZX';c.mesh.rotation.y=-c.a;const cs=Math.cos(c.a)*8,sn=Math.sin(c.a)*8;c.mesh.rotation.z=Math.atan2(gh(c.x+cs,c.y+sn)-gh(c.x-cs,c.y-sn),16)}
function render(k){
 const sp=Math.hypot(p.vx,p.vy);
 if(state=='title')camA+=.004*k;else camA+=((((p.a-camA)+PI*3)%(PI*2))-PI)*.09*k;
 const dist=82+sp*3.5,sx=shake?R(-shake,shake):0;
 cam.position.set(p.x-Math.cos(camA)*dist+sx,44+sp*1.2+gh(p.x,p.y),p.y-Math.sin(camA)*dist+sx);
 cam.lookAt(p.x+Math.cos(camA)*40,6+gh(p.x,p.y),p.y+Math.sin(camA)*40);
 cam.fov=62+sp*1.6;cam.updateProjectionMatrix();
 for(const c of coins){c.mesh.rotation.y+=.06*k;c.mesh.position.set(c.x,7+gh(c.x,c.y)+Math.sin(t*3+c.ph)*1.5,c.y)}
 sync(p);p.mesh.visible=state!='over';
 const on=Math.floor(Date.now()/110)%2;
 for(const c of cars){sync(c);if((c.t=='police'&&!c.wreck)||(c.t=='blk'&&c.mesh.l1)){c.mesh.l1.material=on?LR:LB;c.mesh.l2.material=on?LB:LR}}
 for(const q of peds){animPed(q,k);continue;const m=q.mesh,sp=Math.hypot(q.vx,q.vy);
  if(q.hit>0){const y=Math.sin((1-q.hit/45)*PI)*30;m.position.set(q.x,y+.6+gh(q.x,q.y),q.y);m.rotation.x+=.3*k;m.aL.rotation.z=2.4;m.aR.rotation.z=-2.4;m.lL.rotation.z=.6;m.lR.rotation.z=-.6;continue}
  m.position.set(q.x,.6+gh(q.x,q.y),q.y);
  if(sp>.1){let d=Math.atan2(q.vy,q.vx)-q.fa;d=((d+PI*3)%(PI*2))-PI;q.fa+=d*.25*Math.min(k,1.5);q.ph+=sp*.5*k;const s=Math.sin(q.ph)*(sp>1.5?1:.55);m.lL.rotation.z=s;m.lR.rotation.z=-s;m.aL.rotation.z=-s;m.aR.rotation.z=s}
  else{m.lL.rotation.z=m.lR.rotation.z=m.aL.rotation.z=m.aR.rotation.z=0}
  m.rotation.y=-q.fa}
 for(let i=0;i<MAXP;i++){const q=parts[i];if(q){pp[i*3]=q.x;pp[i*3+1]=q.h;pp[i*3+2]=q.y;pcl[i*3]=q.c.r;pcl[i*3+1]=q.c.g;pcl[i*3+2]=q.c.b}else pp[i*3+1]=-1000}
 pg.attributes.position.needsUpdate=pg.attributes.color.needsUpdate=true;
 updateSignals();renderV2(k);
 ren.render(scene,cam);
 /* HUD */
 g.clearRect(0,0,W,H);g.textAlign='center';g.font='bold 18px sans-serif';g.shadowColor='#000';g.shadowBlur=4;
 for(const q of pops){V.set(q.x,25+(60-q.l)*.6,q.y).project(cam);if(V.z>=1)continue;
  g.globalAlpha=clamp(q.l/30,0,1);g.fillStyle=q.col;g.fillText(q.txt,(V.x*.5+.5)*W,(-V.y*.5+.5)*H)}g.globalAlpha=1;
 g.shadowBlur=0;g.textAlign='left';g.fillStyle='rgba(0,0,0,.45)';g.fillRect(14,14,220,22);
 g.fillStyle=p.hp>.35*p.mhp?'#6be08a':'#ff5a5f';g.fillRect(16,16,216*clamp(p.hp/p.mhp,0,1),18);
 g.font='bold 16px sans-serif';g.fillStyle='#fff';g.fillText('車体',20,31);
 g.shadowBlur=4;g.font='bold 22px sans-serif';g.fillText('スコア '+Math.floor(score),16,64);
 g.fillStyle='#ffd86b';g.fillText('手配 '+'★'.repeat(stars)+'☆'.repeat(5-stars),16,92);
 g.fillStyle='#fff';g.font='bold 16px sans-serif';g.fillText('逃走 '+Math.floor(t)+'秒',16,116);drawHud2();
 if(state!='title'){drawMini();drawCompass()}
 if(combo>1){g.textAlign='right';g.font='bold 28px sans-serif';g.fillStyle='#ffd86b';g.fillText('COMBO ×'+combo,W-20,MM.cy+MM.R+34)}
 g.textAlign='center';g.fillStyle='#fff';
 if(state=='title'){g.shadowBlur=0;g.fillStyle='rgba(10,15,30,.55)';g.fillRect(0,0,W,H);g.fillStyle='#fff';
  g.font='bold 44px sans-serif';g.fillText('🚓 シティ・エスケープ 3D',W/2,H/2-60);
  g.font='20px sans-serif';g.fillText('警察から逃げ切れ！歩行者や車を跳ね飛ばしてスコアを稼ごう',W/2,H/2-14);
  g.fillText('W：アクセル　S：ブレーキ/バック　A・D：ハンドル　S+A/D：ドリフト',W/2,H/2+20);
  g.font='bold 24px sans-serif';g.fillStyle='#ffd86b';g.fillText('Enter キーでスタート',W/2,H/2+70);g.font='bold 18px sans-serif';g.fillStyle='#fff';g.fillText('難易度 [1]イージー  [2]ノーマル  [3]ハード  現在: '+DN[diff],W/2,H/2+104);g.fillText('R：車の復帰（動けなくなったとき）',W/2,H/2+130);g.fillText('E：ミッション開始　0：メニュー（設定）　ガレージで購入・改造・修理',W/2,H/2+156)}
 if(state=='over'){g.shadowBlur=0;g.fillStyle='rgba(10,15,30,.65)';g.fillRect(0,0,W,H);g.fillStyle='#ff7a8a';
  g.font='bold 48px sans-serif';g.fillText(endMsg,W/2,H/2-60);g.fillStyle='#fff';g.font='22px sans-serif';
  g.fillText('スコア '+Math.floor(score)+' ＋ 生存ボーナス '+Math.floor(t*10)+'（'+Math.floor(t)+'秒）',W/2,H/2-10);
  g.fillText('ベスト '+best,W/2,H/2+24);g.font='bold 24px sans-serif';g.fillStyle='#ffd86b';g.fillText('Enter / R でリトライ',W/2,H/2+70)}
 g.shadowBlur=0}
/* ===== v2 systems (late) ===== */
const SAVE_KEY='cityescape3d_v2';
let SAVE={money:0,best:0,owned:['sedan'],cur:'sedan',up:{},paint:{},missions:{},settings:{vol:.6,traffic:1,tod:'auto',weather:'clear',diff:1}};
const VEH=[
 {id:'sedan',n:'ファミリーセダン',cat:'一般車',price:0,kind:0,top:1,acc:1,turn:1,dur:100,col:'#ff5a5f'},
 {id:'hatch',n:'コンパクト',cat:'一般車',price:1800,kind:3,top:.94,acc:1.08,turn:1.12,dur:85,col:'#4fc3e8'},
 {id:'suv',n:'SUV',cat:'SUV',price:4500,kind:1,top:.96,acc:.92,turn:.86,dur:170,col:'#3f6b4a'},
 {id:'pick',n:'ピックアップ',cat:'SUV',price:7000,kind:1,top:1.02,acc:1,turn:.9,dur:150,col:'#e0a030'},
 {id:'coupe',n:'スポーツクーペ',cat:'スポーツカー',price:9500,kind:2,top:1.1,acc:1.22,turn:1.18,dur:90,col:'#d62828'},
 {id:'gt',n:'ターボGT',cat:'スポーツカー',price:19000,kind:2,top:1.2,acc:1.3,turn:1.12,dur:105,col:'#1f4e8c'},
 {id:'hyper',n:'ハイパーカー',cat:'スーパーカー',price:48000,kind:2,top:1.38,acc:1.5,turn:1.22,dur:85,col:'#f2b705'},
 {id:'armor',n:'装甲SUV',cat:'SUV',price:26000,kind:1,top:.9,acc:.85,turn:.8,dur:320,col:'#2b2f36'}];
const PAINTS=['#ff5a5f','#d62828','#e0a030','#f2f2f2','#1f2328','#4fc3e8','#1f4e8c','#2e6b4f','#8a4fb0','#f2b705'];
function loadSave(){try{const raw=localStorage.getItem(SAVE_KEY);if(!raw)return;const s=JSON.parse(raw);if(!s||typeof s!='object')return;
 if(typeof s.money=='number'&&isFinite(s.money))SAVE.money=Math.max(0,s.money|0);if(typeof s.best=='number'&&isFinite(s.best))SAVE.best=s.best;
 if(Array.isArray(s.owned)){const o=s.owned.filter(x=>typeof x=='string'&&VEH.some(v=>v.id==x));if(o.length)SAVE.owned=o}
 if(typeof s.cur=='string'&&SAVE.owned.includes(s.cur))SAVE.cur=s.cur;
 if(s.up&&typeof s.up=='object')SAVE.up=s.up;if(s.paint&&typeof s.paint=='object')SAVE.paint=s.paint;if(s.missions&&typeof s.missions=='object')SAVE.missions=s.missions;
 if(s.settings&&typeof s.settings=='object')Object.assign(SAVE.settings,s.settings)}catch(e){}}
function saveGame(){try{SAVE.money=money;SAVE.best=Math.max(SAVE.best,best);SAVE.settings.diff=diff;localStorage.setItem(SAVE_KEY,JSON.stringify(SAVE))}catch(e){}}
loadSave();best=Math.max(best,SAVE.best);diff=clamp(SAVE.settings.diff|0,0,2);
const vcur=()=>VEH.find(v=>v.id==SAVE.cur)||VEH[0],vup=id=>SAVE.up[id]||(SAVE.up[id]={e:0,t:0,a:0});
const upCost=(v,k,l)=>Math.round((k=='e'?900:k=='t'?700:800)*(l+1)*(1+v.price/40000));
const repairCost=()=>Math.ceil(Math.max(0,p.mhp-p.hp)*6*(1+vcur().price/20000));
function applyVeh(keep){const v=vcur(),u=vup(v.id),col=SAVE.paint[v.id]||v.col,ratio=p.mhp?p.hp/p.mhp:1;
 p.veh=v;p.mhp=v.dur;p.tn=v.turn*(1+.05*u.t);p.lat=.86-.02*u.t;p.stat={term:v.top*(1+.04*u.e),accS:v.acc*(1+.07*u.e)};p.dr=1-.009*p.stat.accS;p.armor=u.a;
 p.hp=keep?Math.max(1,p.mhp*ratio):p.mhp;
 if(p.mesh)scene.remove(p.mesh);p.mesh=carMesh2('p',col,v.kind);scene.add(p.mesh);p.col=col}
/* ---- DOM overlay helpers ---- */
let menuOpen=false,OV=null,gtab='car',garageCd=0,PROMPT='';
function el(tag,a,...kids){const e=document.createElement(tag);if(a)for(const k in a){if(k=='on'){for(const ev in a.on)e.addEventListener(ev,a.on[ev])}else if(k=='text')e.textContent=a[k];else if(k=='style')e.style.cssText=a[k];else if(a[k]!=null)e.setAttribute(k,a[k])}
 for(const c of kids)if(c)e.appendChild(typeof c=='string'?document.createTextNode(c):c);return e}
function closeOv(){try{if(OV&&OV.parentNode)OV.parentNode.removeChild(OV)}catch(e){}OV=null;menuOpen=false}
function ovShow(fn){closeOv();OV=el('div',{id:'ov'});fn(OV);document.body.appendChild(OV);menuOpen=true}
function statRow(label,val,cur,max){const d=val-cur,r=el('div',{class:'st'},el('span',{text:label}),el('div',{class:'bar'},el('i',{style:'width:'+Math.min(100,Math.round(val/max*100))+'%'})));
 if(Math.abs(d)>.004)r.appendChild(el('em',{class:d>0?'up':'dn',text:(d>0?'+':'')+(Math.round(d*100)/100)}));return r}
function renderGarage(root){root.innerHTML='';const box=el('div',{class:'box'});root.appendChild(box);
 box.appendChild(el('div',{class:'hd'},el('b',{text:'🔧 ガレージ'}),el('span',{text:'所持金 $'+money})));
 const tabs=el('div',{class:'tabs'});[['car','車の購入・乗り換え'],['up','改造・塗装'],['rep','修理']].forEach(([k,n])=>tabs.appendChild(el('button',{class:gtab==k?'on':'',text:n,on:{click:()=>{gtab=k;renderGarage(root)}}})));box.appendChild(tabs);
 const body=el('div',{class:'body'});box.appendChild(body);
 if(gtab=='car'){const cu=vcur();for(const v of VEH){const own=SAVE.owned.includes(v.id),cur=v.id==SAVE.cur;
   const card=el('div',{class:'card'},el('div',{class:'row'},el('b',{text:v.n+'  ['+v.cat+']'}),el('span',{text:cur?'使用中':own?'所有済み':'$'+v.price})));
   card.appendChild(statRow('最高速',v.top,cu.top,1.5));card.appendChild(statRow('加速',v.acc,cu.acc,1.5));card.appendChild(statRow('旋回',v.turn,cu.turn,1.3));card.appendChild(statRow('耐久',v.dur/100,cu.dur/100,3.3));
   if(own){if(!cur)card.appendChild(el('button',{text:'この車に乗り換える',on:{click:()=>{SAVE.cur=v.id;applyVeh(true);saveGame();snd('buy');renderGarage(root)}}}))}
   else card.appendChild(el('button',{class:'pri',text:'購入 $'+v.price,disabled:money<v.price?'disabled':null,on:{click:()=>{if(money>=v.price){money-=v.price;SAVE.owned.push(v.id);saveGame();snd('buy');renderGarage(root)}}}}));
   body.appendChild(card)}}
 else if(gtab=='up'){const v=vcur(),u=vup(v.id);body.appendChild(el('div',{class:'row'},el('b',{text:v.n+' の改造'})));
  [['e','エンジン','加速・最高速 +'],['t','タイヤ','旋回・グリップ +'],['a','耐久装甲','被ダメージ -8%/Lv']].forEach(([k,n,ef])=>{const l=u[k],cost=upCost(v,k,l),card=el('div',{class:'card'},el('div',{class:'row'},el('b',{text:n+'  Lv '+l+'/5'}),el('span',{text:ef})));
   card.appendChild(el('button',{class:'pri',text:l>=5?'最大':'強化 $'+cost,disabled:(l>=5||money<cost)?'disabled':null,on:{click:()=>{if(l<5&&money>=cost){money-=cost;u[k]++;applyVeh(true);saveGame();snd('buy');renderGarage(root)}}}}));body.appendChild(card)});
  const pc=el('div',{class:'card'},el('div',{class:'row'},el('b',{text:'外装カラー（$250）'})));const cur=SAVE.paint[v.id]||v.col;
  PAINTS.forEach(c=>pc.appendChild(el('span',{class:'sw',style:'background:'+c+(c==cur?';outline:3px solid #fff':''),on:{click:()=>{if(c==cur)return;if(money>=250){money-=250;SAVE.paint[v.id]=c;applyVeh(true);saveGame();snd('buy');renderGarage(root)}}}})));body.appendChild(pc)}
 else{const cost=repairCost(),pct=Math.round(p.hp/p.mhp*100);
  body.appendChild(el('div',{class:'card'},el('div',{class:'row'},el('b',{text:'車体の状態 '+pct+'%'}),el('span',{text:'修理費 $'+cost})),el('div',{class:'bar'},el('i',{style:'width:'+pct+'%'})),
   el('div',{style:'margin-top:10px'},el('button',{class:'pri',text:cost>0?'修理する $'+cost:'修理の必要なし',disabled:(cost<=0||money<cost)?'disabled':null,on:{click:()=>{if(cost>0&&money>=cost){money-=cost;p.hp=p.mhp;saveGame();snd('buy');renderGarage(root)}}}}))))}
 box.appendChild(el('div',{class:'ft'},el('button',{class:'pri',text:'ガレージを出る',on:{click:()=>{closeOv();garageCd=4;saveGame()}}})))}
function openGarage(){gtab='car';ovShow(renderGarage);if(stars>0)notify('ガレージ（警察は一時停止中）','#8be28b')}
function showResult(title,lines,col){ovShow(root=>{const box=el('div',{class:'box',style:'width:min(520px,92vw)'});root.appendChild(box);box.appendChild(el('div',{class:'hd'},el('b',{text:title,style:'color:'+col})));
 const b=el('div',{class:'body'});lines.forEach(l=>b.appendChild(el('div',{class:'row',text:l})));box.appendChild(b);box.appendChild(el('div',{class:'ft'},el('button',{class:'pri',text:'OK',on:{click:()=>{closeOv()}}})))})}
let confirmReset=false;
function renderPause(root){root.innerHTML='';const box=el('div',{class:'box'});root.appendChild(box);const S=SAVE.settings;
 box.appendChild(el('div',{class:'hd'},el('b',{text:'⏸ ポーズ・設定'}),el('span',{text:'$'+money+'　ベスト '+best})));
 const body=el('div',{class:'body'});box.appendChild(body);
 const choice=(label,key,opts)=>{const row=el('div',{class:'row'},el('span',{text:label}));const g=el('div');opts.forEach(([val,n])=>g.appendChild(el('button',{class:S[key]==val?'on':'',text:n,on:{click:()=>{S[key]=val;if(key=='diff')diff=val;saveGame();renderPause(root)}}})));row.appendChild(g);body.appendChild(row)};
 body.appendChild(el('div',{class:'row'},el('span',{text:'音量'}),el('input',{type:'range',min:0,max:100,value:Math.round(S.vol*100),on:{input:e=>{S.vol=e.target.value/100;setVol();saveGame()}}})));
 choice('交通量','traffic',[[0,'低'],[1,'中'],[2,'高']]);choice('時間帯','tod',[['auto','自動'],['day','昼'],['night','夜']]);choice('天気','weather',[['clear','晴れ'],['rain','雨'],['random','ランダム']]);choice('難易度','diff',[[0,'イージー'],[1,'ノーマル'],[2,'ハード']]);
 body.appendChild(el('div',{class:'row'},el('span',{text:'操作：W加速 S減速/バック A・D旋回 Space・S+A/Dドリフト R復帰 E ミッション開始 0 メニュー'})));
 if(!confirmReset)body.appendChild(el('div',{class:'row'},el('span',{text:'セーブデータ'}),el('button',{text:'データをリセット…',on:{click:()=>{confirmReset=true;renderPause(root)}}})));
 else body.appendChild(el('div',{class:'card'},el('div',{class:'row',text:'本当に所持金・車・改造・記録をすべて削除しますか？（元に戻せません）'}),el('button',{text:'キャンセル',on:{click:()=>{confirmReset=false;renderPause(root)}}}),el('button',{class:'pri',text:'はい、削除する',on:{click:()=>{
  try{localStorage.removeItem(SAVE_KEY)}catch(e){}SAVE={money:0,best:0,owned:['sedan'],cur:'sedan',up:{},paint:{},missions:{},settings:{vol:S.vol,traffic:S.traffic,tod:S.tod,weather:S.weather,diff:S.diff}};money=0;best=0;confirmReset=false;applyVeh(false);saveGame();notify('データを削除しました','#ff9a9a');renderPause(root)}}})));
 box.appendChild(el('div',{class:'ft'},el('button',{class:'pri',text:'再開',on:{click:()=>{closeOv();saveGame()}}})))}
function openPause(){confirmReset=false;ovShow(renderPause)}
addEventListener('keydown',e=>{if((e.code=='Digit0'||e.code=='Numpad0')&&state=='play'){if(menuOpen)closeOv();else openPause();saveGame()}});

/* ---- audio ---- */
let AC=null,AM=null,AU={};
const vol=()=>SAVE.settings.vol;
function setVol(){try{if(AM)AM.gain.value=vol()}catch(e){}}
function initAudio(){if(AC)return;try{const Ctx=window.AudioContext||window.webkitAudioContext;if(!Ctx)return;AC=new Ctx();AM=AC.createGain();AM.connect(AC.destination);setVol();
 const mk=(type,f)=>{const o=AC.createOscillator();o.type=type;o.frequency.value=f;o.start();return o};
 const eo=mk('sawtooth',50),eo2=mk('square',25),lp=AC.createBiquadFilter(),eg=AC.createGain();lp.type='lowpass';lp.frequency.value=500;eo.connect(lp);eo2.connect(lp);lp.connect(eg);eg.connect(AM);eg.gain.value=0;AU.eng={eo,eo2,eg,lp};
 const so=mk('square',800),sg=AC.createGain();so.connect(sg);sg.connect(AM);sg.gain.value=0;AU.sir={so,sg,t:0};
 const nb=AC.createBuffer(1,AC.sampleRate*2,AC.sampleRate),d=nb.getChannelData(0);for(let i=0;i<d.length;i++)d[i]=Math.random()*2-1;AU.nb=nb;
 const rs=AC.createBufferSource();rs.buffer=nb;rs.loop=true;const bp=AC.createBiquadFilter();bp.type='bandpass';bp.frequency.value=2500;const rg=AC.createGain();rg.gain.value=0;rs.connect(bp);bp.connect(rg);rg.connect(AM);rs.start();AU.rain=rg;
 const ho=mk('sawtooth',40),hg=AC.createGain();ho.connect(hg);hg.connect(AM);hg.gain.value=0;AU.heli={ho,hg}}catch(e){AC=null}}
addEventListener('keydown',()=>initAudio());addEventListener('pointerdown',()=>initAudio());
function snd(name){if(!AC)return;try{const now=AC.currentTime,g=AC.createGain();g.connect(AM);
 const tone=(f,t0,dur,type,gain)=>{const o=AC.createOscillator();o.type=type||'sine';o.frequency.setValueAtTime(f,now+t0);const gg=AC.createGain();gg.gain.setValueAtTime(gain,now+t0);gg.gain.exponentialRampToValueAtTime(.001,now+t0+dur);o.connect(gg);gg.connect(AM);o.start(now+t0);o.stop(now+t0+dur+.02)};
 if(name=='hit'){const s=AC.createBufferSource();s.buffer=AU.nb;const lp=AC.createBiquadFilter();lp.type='lowpass';lp.frequency.value=900;const gg=AC.createGain();gg.gain.setValueAtTime(.5,now);gg.gain.exponentialRampToValueAtTime(.001,now+.3);s.connect(lp);lp.connect(gg);gg.connect(AM);s.start(now,Math.random(),.3)}
 else if(name=='coin'){tone(1320,0,.12,'square',.08);tone(1760,.07,.14,'square',.08)}
 else if(name=='star'){tone(880,0,.16,'square',.12);tone(660,.18,.16,'square',.12);tone(880,.36,.2,'square',.12)}
 else if(name=='ok'){[523,659,784,1046].forEach((f,i)=>tone(f,i*.1,.25,'triangle',.18))}
 else if(name=='fail'){[392,330,262,196].forEach((f,i)=>tone(f,i*.14,.3,'sawtooth',.12))}
 else if(name=='buy'){tone(988,0,.1,'triangle',.15);tone(1319,.09,.18,'triangle',.15)}
}catch(e){}}
let audNear=0;
function audioTick(thr,sp){if(!AC)return;try{const on=state=='play'&&!menuOpen,v=vol();
 AU.eng.eo.frequency.value=46+sp*17;AU.eng.eo2.frequency.value=23+sp*8.5;AU.eng.lp.frequency.value=380+sp*70;AU.eng.eg.gain.value=on?(.035+.035*thr)*v:0;
 let nd=1e9;for(const c of cars)if(c.t=='police'&&!c.wreck&&!c.leave){const d=Math.hypot(c.x-p.x,c.y-p.y);if(d<nd)nd=d}
 const sir=on&&stars>0&&nd<650?(1-nd/650):0;AU.sir.t+=1;AU.sir.so.frequency.value=Math.floor(AU.sir.t/26)%2?960:720;AU.sir.sg.gain.value=sir*.05*v;
 AU.rain.gain.value=on?RAINV*.07*v:0;AU.heli.hg.gain.value=on&&HELI?(.03*v*(Math.floor(AU.sir.t/3)%2?1:.4)):0}catch(e){}}
/* ---- day/night & weather ---- */
let TOD=9,RAINV=0,RTARGET=0,wxT=150,GARCD=0;
const SKYD=new THREE.Color(0xbfe3ff),SKYN=new THREE.Color(0x0a1022),SKYR=new THREE.Color(0xff9a6a),SKYG=new THREE.Color(0x7d8896),SKYC=new THREE.Color(0xbfe3ff);
const HEAD=new THREE.SpotLight(0xfff2cc,0,320,.7,.5,1);scene.add(HEAD);scene.add(HEAD.target);
(function(){const em=document.createElement('canvas');em.width=em.height=128;const x=em.getContext('2d');x.fillStyle='#000';x.fillRect(0,0,128,128);for(let i=0;i<8;i++)for(let j=0;j<8;j++){if(Math.random()<.62){x.fillStyle=Math.random()<.5?'#ffd98a':'#fff1c4';x.fillRect(3+i*16,4+j*16,9,9)}}
 if(BM){BM.emissiveMap=new THREE.CanvasTexture(em);BM.emissive=new THREE.Color(0xffffff);BM.emissiveIntensity=0}
 for(const k of['#fff6c8','#ff2a2a']){const m=mat(k);m.emissive=new THREE.Color(k=='#ff2a2a'?0xaa1111:0xffe9a0);m.emissiveIntensity=.8}})();
const RAIN=(function(){const n=900,pos=new Float32Array(n*3),off=[];for(let i=0;i<n;i++)off.push([R(-380,380),R(0,300),R(-380,380)]);
 const g=new THREE.BufferGeometry();g.setAttribute('position',new THREE.BufferAttribute(pos,3));const o=new THREE.Points(g,new THREE.PointsMaterial({size:2.4,color:0xb9d3ee,transparent:true,opacity:.6,depthWrite:false}));o.frustumCulled=false;o.visible=false;scene.add(o);return{o,off,pos,n}})();
function applyEnv(dt,k){const S=SAVE.settings,m=S.tod;
 if(m=='auto')TOD=(TOD+dt*24/600)%24;else TOD=m=='day'?12:0;
 let day=m=='day'?1:m=='night'?0:clamp((Math.sin((TOD-6)/12*PI)+.2)/1.0,0,1);day=day*day*(3-2*day);
 const w=S.weather;if(w=='rain')RTARGET=1;else if(w=='clear')RTARGET=0;else{wxT-=dt;if(wxT<=0){RTARGET=RTARGET>0?0:(Math.random()<.55?1:0);wxT=R(110,240)}}
 RAINV+=(RTARGET-RAINV)*Math.min(1,dt*.35);
 const dusk=m=='auto'?clamp(1-Math.abs(day-.5)*3,0,1)*.6:0;
 SKYC.copy(SKYN).lerp(SKYD,day).lerp(SKYR,dusk*.65).lerp(SKYG,RAINV*.7*(.3+.7*day));
 scene.background.copy(SKYC);scene.fog.color.copy(SKYC);scene.fog.near=520-300*RAINV;scene.fog.far=2100-1000*RAINV-(1-day)*350;
 HL.intensity=.3+.68*day*(1-.3*RAINV);sun.intensity=.04+.72*day*(1-.55*RAINV);sun.color.setRGB(1,.95-dusk*.3,.85-dusk*.5);
 if(BM)BM.emissiveIntensity=(1-day)*.9;
 HEAD.intensity=(1-day)*1.7+RAINV*.35;
 const rd=mat('#555b68').color;rd.setRGB(.333*(1-.38*RAINV),.357*(1-.38*RAINV),.408*(1-.32*RAINV));
 RAIN.o.visible=RAINV>.05;RAIN.o.material.opacity=.15+.5*RAINV;
 if(RAIN.o.visible){const s=RAIN.pos,cx=cam.position.x,cy=cam.position.y,cz=cam.position.z;for(let i=0;i<RAIN.n;i++){const o=RAIN.off[i];o[1]-=16*k;if(o[1]<0)o[1]+=300;s[i*3]=cx+o[0];s[i*3+1]=Math.max(0,cy-120+o[1]);s[i*3+2]=cz+o[2]}RAIN.o.geometry.attributes.position.needsUpdate=true}}
const clockStr=()=>{const hh=Math.floor(TOD),mm=Math.floor((TOD-hh)*60);return(hh<10?'0':'')+hh+':'+(mm<10?'0':'')+mm};
/* ---- 3D markers ---- */
function mkMarker(x,y,col,label,h){const g=new THREE.Group();
 const cyl=new THREE.Mesh(new THREE.CylinderGeometry(30,30,h||70,24,1,true),new THREE.MeshBasicMaterial({color:col,transparent:true,opacity:.28,side:THREE.DoubleSide,depthWrite:false}));cyl.position.y=(h||70)/2;g.add(cyl);
 const ring=new THREE.Mesh(new THREE.TorusGeometry(30,2,8,32),new THREE.MeshBasicMaterial({color:col}));ring.rotation.x=PI/2;ring.position.y=2;g.add(ring);
 if(label){const cv=document.createElement('canvas');cv.width=256;cv.height=64;const c2=cv.getContext('2d');c2.font='bold 38px sans-serif';c2.textAlign='center';c2.lineWidth=7;c2.strokeStyle='#000';c2.strokeText(label,128,46);c2.fillStyle='#fff';c2.fillText(label,128,46);
  const sp=new THREE.Sprite(new THREE.SpriteMaterial({map:new THREE.CanvasTexture(cv),transparent:true,depthTest:false}));sp.scale.set(70,17.5,1);sp.position.y=(h||70)+16;g.add(sp)}
 g.position.set(x,.5,y);scene.add(g);return g}
const GARAGES=[[8,9],[16,8],[24,14],[12,22],[5,28],[27,29],[16,26]].map(([i,j])=>({x:i*CELL+50,y:j*CELL+50})),
 MPOS=[{id:'A',i:12,j:7},{id:'A',i:21,j:22},{id:'B',i:6,j:15},{id:'B',i:26,j:27},{id:'C',i:14,j:3},{id:'C',i:10,j:29}].map(m=>({id:m.id,x:m.i*CELL+50,y:m.j*CELL+50})),
 MNAME={A:'タイムアタック',B:'逃走チャレンジ',C:'長距離ドライブ'},MCOL={A:0x2ad4ff,B:0xff4d5e,C:0xffa63a};
const MARKS=[];
GARAGES.forEach(g=>{g.mesh=mkMarker(g.x,g.y,0x33ff99,'ガレージ',50)});
MPOS.forEach(m=>{m.mesh=mkMarker(m.x,m.y,MCOL[m.id],MNAME[m.id],80)});
const TGT=mkMarker(0,0,0xffe14a,'',120);TGT.visible=false;

let MS=null,visited=new Set(),RBL=[],HELI=null,EVL=[],rbT=0,evT=40;
/* ---- traffic types ---- */
function truckMesh(col){const G=new THREE.Group(),m=mat(col),dk=mat('#222222');
 const cab=bx(G,m,11,10,0,13,13,17),cargo=bx(G,mat(pk2(['#f2f2f2','#d9d9d9','#c9a227','#2e6b4f'])),-8,12,0,32,17,18.5);
 bx(G,mat('#1c2733'),17.4,13,0,.8,6,14);bx(G,dk,0,4,0,46,2,14);
 for(const sx of[-17,-8,13])for(const sz of[-8.6,8.6]){const w=new THREE.Mesh(gCyl,dk);w.rotation.x=PI/2;w.scale.set(3.8,2.6,3.8);w.position.set(sx,3.8,sz);G.add(w)}
 for(const z of[-5.8,5.8]){bx(G,mat('#fff6c8'),17.6,6,z,.8,1.8,3.4);bx(G,mat('#ff2a2a'),-24.2,6,z,.8,1.8,3.4)}
 G.tint=[cab,cargo];G.rotation.order='YZX';const s=new THREE.Mesh(new THREE.PlaneGeometry(52,24),SHM);s.rotation.x=-PI/2;s.position.y=.5;G.add(s);return G}
const TTYPES=[{w:.34,kind:0,sp:[1.4,2.2]},{w:.1,kind:0,sp:[1.5,2.2],opt:'taxi',col:'#f2c230'},{w:.12,kind:2,sp:[1.9,2.6]},{w:.2,kind:1,sp:[1.3,2]},{w:.12,kind:3,sp:[1.4,2.1]},{w:.12,kind:4,sp:[1,1.5]}];
function pickType(){let r=Math.random(),tp=TTYPES[0];for(const q of TTYPES){if(r<q.w){tp=q;break}r-=q.w}return{kind:tp.kind,opt:tp.opt,sp:tp.sp,col:()=>tp.col||CARC[Math.floor(R(0,CARC.length))]}}
function spawnTraffic(a0=650,a1=1050){const s=spawnPt(a0,a1),d=s.h?(Math.random()<.5?0:2):(Math.random()<.5?1:3);
 if(s.h)s.y+=laneOff(d);else s.x+=laneOff(d);if(Math.hypot(s.x-p.x,s.y-p.y)<120)return;
 const tp=pickType(),c=mkCar(s.x,s.y,d*PI/2,'t',tp.col(),tp.kind,tp.opt);c.d=d;c.sp=R(tp.sp[0],tp.sp[1]);cars.push(c)}
function spawnFw(){const x=clamp(p.x+R(-1400,1400),150,SIZE-150),east=Math.random()<.5,lane=Math.floor(R(0,3)),d=east?0:2,z=FWZ0+(east?36+lane*38:188+lane*38);
 if(Math.hypot(x-p.x,z-p.y)<260)return;for(const o of cars)if(o.fw&&Math.abs(o.y-z)<10&&Math.abs(o.x-x)<90)return;
 const tp=pickType(),c=mkCar(x,z,d*PI/2,'t',tp.col(),tp.kind,tp.opt);c.d=d;c.fw=1;c.sp=R(3.4,4.7)*(tp.kind==4?.75:1);c.cs=c.sp;c.laneZ=z;cars.push(c)}
function driveF(c,k){let tg=c.sp;const inc=c.d==0;
 for(const o of cars){if(o===c||(!o.fw&&!o.free)||Math.abs(o.y-c.y)>14)continue;const gap=(inc?1:-1)*(o.x-c.x);if(gap>0&&gap<110)tg=Math.min(tg,clamp((gap-40)/60,0,1)*c.sp)}
 c.cs+=(tg-c.cs)*.1*k;c.a=c.d*PI/2;c.vx=Math.cos(c.a)*c.cs;c.vy=0;c.y+=(c.laneZ-c.y)*.1*k;c.x+=c.vx*k;if(c.x<-80||c.x>SIZE+80)c.dead=1}
/* ---- roadblocks ---- */
function rbPlace(){const sp=Math.hypot(p.vx,p.vy);if(sp<1.2||mInfo(p.x,p.y)||p.x>SIZE-100||bandF(p.y))return;
 const hx=p.vx/sp,hy=p.vy/sp,horiz=Math.abs(hx)>Math.abs(hy),D=R(560,880);let x=p.x+hx*D,y=p.y+hy*D;
 const mid=v=>{const m=((v%CELL)+CELL)%CELL;return v-m+clamp(m,RW+70,CELL-70)};
 if(horiz){y=clamp(Math.round((y-50)/CELL),1,N-2)*CELL+50;x=mid(x)}else{x=clamp(Math.round((x-50)/CELL),1,N-2)*CELL+50;y=mid(y)}
 if(horiz?hClosed(x,Math.round((y-50)/CELL)):vClosed(y,Math.round((x-50)/CELL)))return;
 if(inCover(x,y)||bandF(y)||Math.hypot(x-p.x,y-p.y)<480)return;
 const g={x,y,hx,hy,age:0,cars:[]};
 for(const o of[-34,0,34]){const c=mkCar(horiz?x:x+o,horiz?y+o:y,horiz?PI/2:0,'police','#fff');c.t='blk';c.vx=c.vy=0;cars.push(c);g.cars.push(c)}
 RBL.push(g)}
function rbRemove(g){for(const c of g.cars){kill(c);const i=cars.indexOf(c);if(i>=0)cars.splice(i,1)}}
function rbUpdate(dt,k){const want=[0,0,0,1,2,3][stars];rbT-=dt;
 for(let i=RBL.length-1;i>=0;i--){const g=RBL[i],d=Math.hypot(g.x-p.x,g.y-p.y);g.age+=dt;
  if(d>1700||g.age>75||(g.age>8&&d>380&&(g.x-p.x)*g.hx+(g.y-p.y)*g.hy<-220)||(stars<3&&g.age>4)){rbRemove(g);RBL.splice(i,1)}}
 if(stars>=3&&RBL.length<want&&rbT<=0&&POL.state=='chase'){rbPlace();rbT=14}}
/* ---- helicopter ---- */
function heliMk(){const G=new THREE.Group(),dk=mat('#23272f'),bl=mat('#2a3f66');
 bx(G,bl,0,0,0,24,9,9);bx(G,bl,-18,2,0,22,3,3);bx(G,dk,-28,5,0,2,9,1.2);bx(G,mat('#9fd4ff'),8,1,0,8,6,8.6);bx(G,dk,0,-7,-5,22,1,1);bx(G,dk,0,-7,5,22,1,1);
 const rot=new THREE.Group();rot.position.y=6;bx(rot,dk,0,0,0,52,.6,3);bx(rot,dk,0,0,0,3,.6,52);G.add(rot);G.rot=rot;G.scale.setScalar(1.4);return G}
let BEAM=null;
function heliUpdate(dt,k){const want=stars>=4&&!(MS&&MS.type=='X');
 if(want&&!HELI){const a=Math.random()*PI*2,hm=heliMk();scene.add(hm);HELI={x:p.x+Math.cos(a)*950,y:p.y+Math.sin(a)*950,h:150,vx:0,vy:0,a:0,t:0,mesh:hm,leave:0,sees:false};notify('警察ヘリが接近中！','#7fb8ff')}
 if(!HELI)return;
 if(!BEAM){const g=new THREE.ConeGeometry(1,1,16,1,true);g.translate(0,-.5,0);g.rotateX(-PI/2);BEAM=new THREE.Mesh(g,new THREE.MeshBasicMaterial({color:0xfff2b0,transparent:true,opacity:.18,side:THREE.DoubleSide,depthWrite:false}));scene.add(BEAM)}
 const H=HELI;H.t+=dt;if(!want)H.leave+=dt;
 let tx=p.x+Math.cos(H.t*.45)*110,ty=p.y+Math.sin(H.t*.45)*110;if(H.leave>0){tx=H.x+(H.x-p.x)*5;ty=H.y+(H.y-p.y)*5}
 const dx=tx-H.x,dy=ty-H.y,d=Math.hypot(dx,dy)||1,sp=Math.min(H.leave>0?9:8,d*.03+Math.hypot(p.vx,p.vy)*.6);
 H.vx+=(dx/d*sp-H.vx)*.04*k;H.vy+=(dy/d*sp-H.vy)*.04*k;H.x+=H.vx*k;H.y+=H.vy*k;if(Math.hypot(H.vx,H.vy)>.5)H.a=Math.atan2(H.vy,H.vx);
 const gy=gh(p.x,p.y);H.mesh.position.set(H.x,H.h+gy*.3,H.y);H.mesh.rotation.y=-H.a;H.mesh.rot.rotation.y+=1.1*k;
 const showB=H.sees&&H.leave==0;BEAM.visible=showB;if(showB){const dz=H.h+gy*.3;BEAM.position.set(H.x,dz-3,H.y);BEAM.lookAt(p.x,gy,p.y);BEAM.scale.set(22,22,Math.hypot(H.x-p.x,H.y-p.y,dz-gy))}
 if(H.leave>7||Math.hypot(H.x-p.x,H.y-p.y)>2600)killHeli()}
function killHeli(){if(HELI){scene.remove(HELI.mesh);HELI=null}if(BEAM)BEAM.visible=false}
/* ---- road-work events ---- */
function segKey(ax,ay,bx_,by_){const a=nodeOf(ax,ay),b=nodeOf(bx_,by_);return(a[0]<b[0]||(a[0]==b[0]&&a[1]<b[1]))?a+'|'+b:b+'|'+a}
let SIGN_TEX=null;
function barMesh(){const G=new THREE.Group();bx(G,mat('#ff7a1a'),0,5,0,5,10,32);bx(G,mat('#f4f4f4'),0,7.5,0,5.2,2.2,32.4);bx(G,mat('#f4f4f4'),0,2.5,0,5.2,2.2,32.4);bx(G,mat('#ffcc33'),0,11.5,0,3,2,3);G.tint=[];return G}
function signMesh(){if(!SIGN_TEX){const cv=document.createElement('canvas');cv.width=256;cv.height=160;const c2=cv.getContext('2d');c2.fillStyle='#ffb21a';c2.fillRect(0,0,256,160);c2.strokeStyle='#222';c2.lineWidth=8;c2.strokeRect(6,6,244,148);c2.fillStyle='#222';c2.font='bold 44px sans-serif';c2.textAlign='center';c2.fillText('工事中',128,70);c2.font='bold 30px sans-serif';c2.fillText('通行止め',128,122);SIGN_TEX=new THREE.CanvasTexture(cv)}
 const G=new THREE.Group();bx(G,mat('#666666'),0,6,0,1.2,12,1.2);const pl=new THREE.Mesh(new THREE.PlaneGeometry(14,9),new THREE.MeshBasicMaterial({map:SIGN_TEX,side:THREE.DoubleSide}));pl.position.y=15;G.add(pl);return G}
function evPlace(){const sn=nodeOf(p.x,p.y);for(let tr=0;tr<20;tr++){const ni=clamp(sn[0]+Math.floor(R(-5,6)),1,N-2),nj=clamp(sn[1]+Math.floor(R(-5,6)),1,N-2),vert=Math.random()<.5,mi=vert?ni:ni+1,mj=vert?nj+1:nj;
  if(mi>N-2||mj>N-2)continue;const ax=ni*CELL+50,ay=nj*CELL+50,bx2=mi*CELL+50,by2=mj*CELL+50;if(segClosed(ax,ay,bx2,by2))continue;
  if(vert&&(nj==FWJ||nj+1==FWJ+1||nj==FWJ-1||nj==FWJ+1))continue;if(!vert&&(nj==FWJ||nj==FWJ+1))continue;
  const cx=(ax+bx2)/2,cy=(ay+by2)/2;if(Math.hypot(cx-p.x,cy-p.y)<700||inCover(cx,cy))continue;
  const e={x:cx,y:cy,age:0,cars:[],signs:[],key:segKey(ax,ay,bx2,by2),warned:false};
  for(const o of[-34,0,34]){const c={x:vert?cx+o:cx,y:vert?cy:cy+o,a:vert?0:PI/2,vx:0,vy:0,t:'blk',col:'#fff',hp:100,acc:0,free:false,wreck:false,life:0,d:0,sp:0,li:null,stuck:0,rev:0,hitCd:0,mesh:barMesh()};scene.add(c.mesh);cars.push(c);e.cars.push(c)}
  for(const s of[-1,1]){const sm=signMesh();sm.position.set(vert?cx:cx+s*190,0,vert?cy+s*190:cy);scene.add(sm);e.signs.push(sm)}
  DYNSEG.add(e.key);EVL.push(e);return}}
function evRemove(e){for(const c of e.cars){kill(c);const i=cars.indexOf(c);if(i>=0)cars.splice(i,1)}for(const s of e.signs)scene.remove(s);DYNSEG.delete(e.key)}
function evUpdate(dt){evT-=dt;for(let i=EVL.length-1;i>=0;i--){const e=EVL[i];e.age+=dt;const d=Math.hypot(e.x-p.x,e.y-p.y);
  if(!e.warned&&d<650){e.warned=true;notify('前方で工事中・通行止め','#ffb347','迂回してください')}
  if(e.age>170||d>2900){evRemove(e);EVL.splice(i,1)}}
 if(evT<=0&&EVL.length<2){evT=55;evPlace()}}
/* ---- missions ---- */
function randNodeFrom(i,j,minH,maxH){for(let tr=0;tr<40;tr++){const ti=clamp(i+Math.floor(R(-maxH,maxH+1)),1,N-2),tj=clamp(j+Math.floor(R(-maxH,maxH+1)),1,N-2),hh=Math.abs(ti-i)+Math.abs(tj-j);if(hh<minH||hh>maxH)continue;const path=navPath(i,j,ti,tj);if(path&&path.length>1)return[ti,tj,path.length-1]}return null}
function startMission(m){if(MS)return;const type=m.id,ni=Math.round((m.x-50)/CELL),nj=Math.round((m.y-50)/CELL);
 const ms={type,name:MNAME[type],t0:t,idx:0,cps:[],hp0:p.hp,sx:m.x,sy:m.y,limit:0,len:0,par:0};
 if(type=='A'||type=='C'){let ci=ni,cj=nj,sum=0;const n=type=='A'?6:5,minH=type=='A'?3:8,maxH=type=='A'?6:14;
  for(let q=0;q<n;q++){const r=randNodeFrom(ci,cj,minH,maxH)||randNodeFrom(ci,cj,2,20);if(!r)break;ms.cps.push({x:r[0]*CELL+50,y:r[1]*CELL+50});sum+=r[2]*CELL;ci=r[0];cj=r[1]}
  if(ms.cps.length<2){notify('ミッションを作れませんでした','#ff9a9a');return}
  ms.len=sum;if(type=='A')ms.limit=Math.round(sum/(4.4*60)*[1.7,1.4,1.2][diff]+14);else{ms.par=Math.round(sum/(3.8*60)*1.1+20);ms.limit=Math.round(sum/(3.8*60)*2.4+40)}}
 else{ms.limit=[90,120,150][diff];heat=Math.max(heat,TH[3])}
 MS=ms;notify(MS.name+' 開始！','#7fe0ff',type=='B'?(ms.limit+'秒間、逃げ切れ！'):(ms.cps.length+'か所を順に通過'));snd('star')}
function missionRewardSave(type,ok,time){const r=SAVE.missions[type]||(SAVE.missions[type]={clears:0,best:0});if(ok){r.clears++;if(!r.best||time<r.best)r.best=Math.round(time)}saveGame()}
function finishMission(ok,why){const m=MS,el=t-m.t0;let reward=0,lines=[];
 if(ok){if(m.type=='A'){const left=Math.max(0,m.limit-el);reward=Math.round(500+left*25+m.cps.length*100);lines=['クリアタイム '+el.toFixed(1)+'秒（制限 '+m.limit+'秒）','残り時間ボーナス '+Math.round(left*25)]}
  else if(m.type=='B'){reward=Math.round(1200+POL.maxS*300+(p.hp/p.mhp)*400);lines=['逃げ切った時間 '+m.limit+'秒','最大手配レベル ★'+POL.maxS]}
  else{const fast=Math.max(0,m.par-el);reward=Math.round(800+fast*15+(p.hp/p.mhp)*300+m.len/20);lines=['走行距離 約'+Math.round(m.len/7.5)+'m','所要時間 '+el.toFixed(1)+'秒（目標 '+m.par+'秒）','車体の状態 '+Math.round(p.hp/p.mhp*100)+'%']}
  money+=reward;score+=reward;lines.push('報酬 $'+reward);snd('ok');missionRewardSave(m.type,true,el);MS=null;TGT.visible=false;showResult('✔ ミッション成功：'+m.name,lines,'#6be08a')}
 else{snd('fail');MS=null;TGT.visible=false;showResult('✖ ミッション失敗：'+m.name,[why||'失敗しました','何度でも再挑戦できます'],'#ff7a7a')}}
function missionUpdate(dt){if(!MS)return;const el=t-MS.t0;
 if(MS.type=='B'){heat=Math.max(heat,TH[3]);POL.searchT=0;TGT.visible=false;if(el>=MS.limit)finishMission(true)}
 else{const cp=MS.cps[MS.idx];TGT.visible=true;TGT.position.set(cp.x,.5,cp.y);TGT.rotation.y+=.03;
  if(Math.hypot(cp.x-p.x,cp.y-p.y)<52){snd('coin');MS.idx++;if(MS.idx>=MS.cps.length)finishMission(true);else notify('チェックポイント '+MS.idx+'/'+MS.cps.length,'#7fe0ff')}
  else if(el>MS.limit)finishMission(false,MS.type=='A'?'時間切れ':'制限時間を大幅に超過')}}
function failMission(msg){const m=MS;MS=null;TGT.visible=false;snd('fail');p.x=m.sx;p.y=m.sy;p.vx=p.vy=0;p.hp=Math.max(p.hp,p.mhp*.5);bustT=0;heat=0;stars=0;POL.state='idle';POL.maxS=0;inv=60;
 for(const c of cars)if(c.t=='police')c.leave=true;showResult('✖ ミッション失敗：'+m.name,[msg+'（開始地点に戻ります）','何度でも再挑戦できます'],'#ff7a7a')}
function missionLines(){if(!MS)return[];const el=t-MS.t0;
 if(MS.type=='B')return['残り '+Math.max(0,Math.ceil(MS.limit-el))+'秒','手配レベル ★'+stars];
 const cp=MS.cps[MS.idx];const d=Math.round(Math.hypot(cp.x-p.x,cp.y-p.y)/7.5);
 return[(MS.type=='A'?'残り '+Math.max(0,Math.ceil(MS.limit-el))+'秒':'経過 '+Math.floor(el)+'秒（目標 '+MS.par+'秒）'),'チェックポイント '+(MS.idx+1)+'/'+MS.cps.length,'次の目的地まで '+d+'m']}

/* ---- core overrides ---- */
function stepCar(c,thr,st,hb,k){
 const ca=Math.cos(c.a),sa=Math.sin(c.a);let f=c.vx*ca+c.vy*sa,l=-c.vx*sa+c.vy*ca;
 f+=thr*c.acc*k;f*=Math.pow(c.dr||.991,k);if(thr==0&&!hb)f*=Math.pow(.993,k);
 l*=Math.pow(hb?.97:Math.min(.95,(c.lat||.86)+.03*RAINV),k);if(hb)f*=Math.pow(.985,k);
 c.a+=st*.055*(c.tn||1)*clamp(f/3,-1,1)*(1-.42*clamp(Math.abs(f)/7,0,1))*k;
 const c2=Math.cos(c.a),s2=Math.sin(c.a);c.vx=f*c2-l*s2;c.vy=f*s2+l*c2;c.x+=c.vx*k;c.y+=c.vy*k}
function hurt(d){if(inv>0||d<=0)return;d*=(1-.08*(p.armor||0));p.hp-=d;inv=8;shake=Math.max(shake,Math.min(10,d*.5));snd('hit')}
function endGame(msg){if(state!='play')return;if(MS){failMission(msg=='BUSTED!'?'逮捕された':'車が大破した');return}
 state='over';endMsg=msg;burst(p.x,p.y,'#ff9a4a',30);const tot=Math.floor(score+t*10);if(tot>best)best=tot;saveGame();snd('fail')}
function collideBlocks(c,r){let imp=0;const ci=Math.floor(c.x/CELL),cj=Math.floor(c.y/CELL),air=ovH(c.x,c.y)>9;
 const hit=(x0,y0,x1,y1)=>{if(c.x<x0-r||c.x>x1+r||c.y<y0-r||c.y>y1+r)return;const qx=clamp(c.x,x0,x1),qy=clamp(c.y,y0,y1);let dx=c.x-qx,dy=c.y-qy,d=Math.hypot(dx,dy);
  if(d<r){if(d<.001){dx=0;dy=-1;d=1}const nx=dx/d,ny=dy/d;c.x=qx+nx*r;c.y=qy+ny*r;const vn=c.vx*nx+c.vy*ny;if(vn<0){c.vx-=1.5*vn*nx;c.vy-=1.5*vn*ny;imp=Math.max(imp,-vn)}}};
 for(let i=ci-1;i<=ci+1;i++)for(let j=cj-1;j<=cj+1;j++){if(i<0||j<0||i>=N||j>=N)continue;const kd=blockKind(i,j),x0=i*CELL+RW+12,y0=j*CELL+RW+12;
  if(ALLEYK[kd]){hit(x0,y0,x0+118,y0+276);hit(x0+158,y0,x0+276,y0+276)}else if(!NOCOL[kd])hit(x0,y0,x0+276,y0+276);
  const pl=PILL.get(i+','+j);if(pl)for(const q of pl)hit(q[0],q[1],q[2],q[3])}
 for(const q of CL)hit(q[0],q[1],q[2],q[3]);
 if(!air&&Math.abs(c.y-(FWZ0+150))<170)for(const q of FWB)hit(q[0],q[1],q[2],q[3]);
 imp=Math.max(imp,mRails(c,r));
 if(c.x<XMIN){c.x=XMIN;c.vx=Math.abs(c.vx)*.5}if(c.x>XMAX-60){c.x=XMAX-60;c.vx=-Math.abs(c.vx)*.5}
 if(c.y<60){c.y=60;c.vy=Math.abs(c.vy)*.5}if(c.y>SIZE-60){c.y=SIZE-60;c.vy=-Math.abs(c.vy)*.5}
 return imp}
function polUpdate(dt,k){
 if(recT>0)recT-=k;
 const act=[];for(const c of cars)if(c.t=='police'&&!c.wreck)act.push(c);
 const live=act.filter(c=>!c.leave).sort((a,b)=>Math.hypot(a.x-p.x,a.y-p.y)-Math.hypot(b.x-p.x,b.y-p.y));live.forEach((c,i)=>c.role=i);
 const cov=inCover(p.x,p.y),sR=cov?90:SIGHT[diff];let seen=false;
 for(const c of live){c.sees=Math.hypot(c.x-p.x,c.y-p.y)<sR&&los(c.x,c.y,p.x,p.y);if(c.sees)seen=true}
 if(HELI){HELI.sees=HELI.leave==0&&!cov&&Math.hypot(HELI.x-p.x,HELI.y-p.y)<720;if(HELI.sees)seen=true}
 POL.seen=seen;
 if(stars>0){
  if(seen){if(POL.state=='search')notify('見つかった！','#ff6b6b');POL.state='chase';POL.lx=p.x;POL.ly=p.y;POL.lvx=p.vx;POL.lvy=p.vy;POL.lostT=0;POL.searchT=0;POL.time+=dt;if(stars<5)heat+=dt*.9;score+=dt*1.5}
  else{POL.lostT+=dt;if(POL.state=='chase'&&POL.lostT>2.5){POL.state='search';notify('警察が捜索中',cov?'#ffd86b':'#ffd86b',cov?'隠れ場所に潜伏中…入口付近を捜索している':'見つからないように逃げ続けよう')}
   if(POL.state=='search'){POL.searchT+=dt;if(POL.searchT>=CD[stars]*CDM[diff]*(cov?1.25:1))lowerStar()}}}
 else{heat=Math.max(0,heat-dt*1.5);POL.state='idle'}
 let sN=0;for(let i=1;i<=5;i++)if(heat>=TH[i])sN=i;
 if(sN>stars){const pr=stars;stars=sN;POL.maxS=Math.max(POL.maxS,sN);POL.state='chase';POL.lostT=0;POL.searchT=0;if(pr==0){POL.lx=p.x;POL.ly=p.y;POL.time=0}starUp(pr);snd('star')}
 const want=CNT[stars];
 if(stars==0){for(const c of act)c.leave=true}
 else{for(let i=want;i<live.length;i++)live[i].leave=true;spawnT-=k;if(live.length<want&&spawnT<=0&&POL.state!='search'){spawnCop(700,1000);spawnT=70}}
 for(let i=cars.length-1;i>=0;i--){const c=cars[i];if(c.t=='police'&&c.leave&&!c.wreck){const d=Math.hypot(c.x-p.x,c.y-p.y);if(d>520||(d>250&&!los(c.x,c.y,p.x,p.y))){kill(c);cars.splice(i,1)}}}
 const near=live.filter(c=>Math.hypot(c.x-p.x,c.y-p.y)<62).length,ps=Math.hypot(p.vx,p.vy);
 if(stars>0&&near>0&&(ps<.9||(!K.KeyW&&!K.KeyS&&!K.Space&&ps<2.6)))bustT+=dt*(near>1?1:.6);else bustT=Math.max(0,bustT-dt*1.5);
 if(bustT>3.5)endGame('BUSTED!')}
let lastThr=0,svT=0;
function update(dt){
 const k=Math.min(dt*60,2.5);t+=dt;lvl=stars;if(garageCd>0)garageCd-=dt;
 if(K.KeyR){K.KeyR=0;recoverPlayer()}
 polUpdate(dt,k);if(state!='play')return;rbUpdate(dt,k);heliUpdate(dt,k);evUpdate(dt);missionUpdate(dt);
 if(inv>0)inv-=k;if(shake>0)shake*=.9;comboT-=k;if(comboT<=0)combo=0;
 let thr=0,st=(K.KeyD?1:0)-(K.KeyA?1:0);if(K.KeyW)thr=1;lastThr=thr;
 const f=p.vx*Math.cos(p.a)+p.vy*Math.sin(p.a),drift=(K.KeyS&&st!=0&&f>2.5)||!!K.Space;
 if(K.KeyS&&!drift){if(f>.8){const bk=Math.pow(.94+.012*RAINV,k);p.vx*=bk;p.vy*=bk}else thr=-.6}
 p.acc=.0616*p.stat.term*p.stat.accS*(.55+.45*clamp(p.hp/(.6*p.mhp),0,1));
 stepCar(p,thr,drift?st*1.5:st,drift,k);
 if(drift&&Math.hypot(p.vx,p.vy)>2){for(const s of[-8,8])addSkid(p.x-Math.cos(p.a)*10-Math.sin(p.a)*s,p.y-Math.sin(p.a)*10+Math.cos(p.a)*s,p.a);smoke(p.x-Math.cos(p.a)*14,p.y-Math.sin(p.a)*14)}
 const ip=collideBlocks(p,15);if(ip>2.5){hurt((ip-2)*2.4);burst(p.x,p.y,'#fff',6)}
 if(p.hp<.35*p.mhp&&Math.random()<.3)smoke(p.x,p.y);
 for(const c of cars){
  if(c.hitCd>0)c.hitCd-=k;
  if(c.t=='blk'){c.vx=c.vy=0;continue}
  if(c.t=='police'){if(c.wreck){c.life-=k;stepCar(c,0,0,true,k);collideBlocks(c,15);if(Math.random()<.4)smoke(c.x,c.y)}else{policeAI(c,k);collideBlocks(c,15)}}
  else if(c.free){c.life+=k;stepCar(c,0,0,true,k);collideBlocks(c,15)}
  else if(c.fw)driveF(c,k);
  else{driveT(c,k);if(c.cs<.15){c.stk=(c.stk||0)+k;if(c.stk>1500)c.dead=1}else c.stk=0}}
 const all=[p,...cars];
 for(let i=0;i<all.length;i++)for(let j=i+1;j<all.length;j++){const a=all[i],b=all[j];if(Math.abs(a.x-b.x)<32&&Math.abs(a.y-b.y)<32)carHit(a,b)}
 const sp=Math.hypot(p.vx,p.vy);
 for(let i=peds.length-1;i>=0;i--){const q=peds[i];
  if(q.hit>0){q.x+=q.vx*k;q.y+=q.vy*k;q.vx*=.94;q.vy*=.94;q.hit-=k;if(q.hit<=0){kill(q);peds.splice(i,1)}continue}
  const dx=q.x-p.x,dy=q.y-p.y,d=Math.hypot(dx,dy);
  if(q.mode!=3&&d<19&&sp>1.5&&Math.abs(gh(q.x,q.y)-gh(p.x,p.y))<10){q.hit=45;addCoin(q.x+R(-10,10),q.y+R(-10,10),50);crime(12);q.vx=p.vx*1.3;q.vy=p.vy*1.3;combo++;comboT=120;const s=100*Math.min(combo,8);score+=s;pop(q.x,q.y,'+'+s,'#fff');burst(q.x,q.y,q.col,6);continue}
  pedAI(q,k,dx,dy,d,sp)}
 const tcap=[12,20,32][SAVE.settings.traffic|0],fwTarget=Math.abs(p.y-(FWZ0+150))<1300?[8,14,22][SAVE.settings.traffic|0]:0;let tc=0,fc=0;
 for(let i=cars.length-1;i>=0;i--){const c=cars[i],d=Math.hypot(c.x-p.x,c.y-p.y);
  if(c.t=='t'){if(c.fw)fc++;else tc++}
  if(c.t=='blk')continue;
  if(c.dead||d>1600||(c.t=='t'&&c.free&&c.life>600)||(c.t=='police'&&c.wreck&&c.life<=0)){if(c.t=='t'){if(c.fw)fc--;else tc--}kill(c);cars.splice(i,1)}}
 for(let i=coins.length-1;i>=0;i--){const c=coins[i],d=Math.hypot(c.x-p.x,c.y-p.y);
  if(d<24&&Math.abs(gh(c.x,c.y)-gh(p.x,p.y))<18){score+=c.val;money+=c.val>=500?c.val:Math.round(c.val/5);pop(c.x,c.y,'💰 +'+c.val,'#ffd84a');burst(c.x,c.y,'#ffd84a',8);snd('coin');kill(c);coins.splice(i,1);continue}
  if(d>1700&&!c.keep){kill(c);coins.splice(i,1)}}
 while(coins.length<150)spawnCoin();while(peds.length<60)spawnPed();
 if(tc<tcap)spawnTraffic();if(fc<fwTarget)spawnFw();
 for(let i=parts.length-1;i>=0;i--){const q=parts[i];q.x+=q.vx*k;q.y+=q.vy*k;q.h=Math.max(0,q.h+q.vh*k);q.vh-=q.g*k;q.l-=k;if(q.l<=0)parts.splice(i,1)}
 for(let i=pops.length-1;i>=0;i--){pops[i].l-=k;if(pops[i].l<=0)pops.splice(i,1)}
 const key=Math.floor(p.x/150)+','+Math.floor(p.y/150);if(!visited.has(key)){visited.add(key);if(sp>1.5)score+=2}
 PROMPT='';
 if(!menuOpen&&!MS){
  for(const g of GARAGES){if(Math.hypot(g.x-p.x,g.y-p.y)<42&&sp<3.4&&garageCd<=0){openGarage();break}}
  for(const m of MPOS){if(Math.hypot(m.x-p.x,m.y-p.y)<46){PROMPT='E キー：'+MNAME[m.id]+'を開始';if(K.KeyE){K.KeyE=0;startMission(m)}break}}}
 svT+=dt;if(svT>8){svT=0;saveGame()}
 if(p.hp<=0){p.hp=0;endGame('WRECKED!')}}
function start(){
 if(cars){cars.forEach(kill);peds.forEach(kill);coins.forEach(kill);kill(p);for(const g of RBL.slice())rbRemove(g);RBL.length=0;for(const e of EVL.slice())evRemove(e);EVL.length=0;killHeli()}
 closeOv();MS=null;TGT.visible=false;visited=new Set();
 p=mkCar(12*CELL+50,12*CELL+50,0,'p','#ff5a5f');applyVeh(false);
 cars=[];peds=[];coins=[];parts=[];pops=[];t=0;score=0;combo=0;comboT=0;shake=0;lvl=1;spawnCd=0;inv=0;stars=1;heat=25;money=SAVE.money;bustT=0;notes=[];flash=0;spawnT=0;recT=0;rbT=8;evT=45;
 POL={state:'chase',lx:p.x,ly:p.y,lvx:0,lvy:0,lostT:-6,searchT:0,seen:false,time:0,maxS:1};endMsg='BUSTED!';SAVE.settings.diff=diff;state='play';
 for(let i=0;i<20;i++)spawnTraffic(150,1100);
 for(let i=0;i<60;i++)spawnPed(100,1200);
 spawnCop(450,650);
 for(let i=0;i<70;i++)spawnCoin(100,1300);
 addSpiralCoins();saveGame()}

/* ---- render hooks & HUD ---- */
function areaName(x,y){if(x>SIZE)return '山岳地帯';if(x<-5)return 'ビーチ（海）';if(bandF(y))return '高速道路';const i=Math.floor(x/CELL),j=Math.floor(y/CELL);
 if(i==0)return 'ビーチ';if(j>=24&&i>=12)return j>=N-1?'港':'工業地帯';return Math.hypot(i-N/2,j-N/2)/(N/2)<.35?'ダウンタウン':'住宅街'}
function renderV2(k){applyEnv(1/60*k,k);
 const ca=Math.cos(p.a),sa=Math.sin(p.a),gy=gh(p.x,p.y);HEAD.position.set(p.x+ca*18,14+gy,p.y+sa*18);HEAD.target.position.set(p.x+ca*130,gy,p.y+sa*130);
 const sp=Math.hypot(p.vx,p.vy);audioTick(lastThr,sp);
 for(const g of GARAGES)g.mesh.children[1].rotation.z+=.04*k;for(const m of MPOS)m.mesh.children[1].rotation.z+=.04*k}
function drawHud2(){g.save();g.shadowBlur=4;g.shadowColor='#000';g.textAlign='left';g.font='bold 18px sans-serif';g.fillStyle='#8be28b';g.fillText('$ '+money,16,140);
 const left=Math.max(0,Math.ceil(CD[stars]*CDM[diff]-POL.searchT)),st=stars==0?['手配なし','#cfd8dc']:POL.state=='search'?['捜索中 '+left+'s','#ffd86b']:['追跡中','#ff6b6b'];
 g.fillStyle=st[1];g.fillText(st[0]+(HELI&&HELI.leave==0?'  🚁':''),16,164);
 g.fillStyle='#bfe3ff';g.font='bold 15px sans-serif';g.fillText(areaName(p.x,p.y)+'   '+clockStr()+(RAINV>.3?'  ☔':''),16,188);
 g.textAlign='center';g.font='bold 42px sans-serif';g.fillStyle='#fff';g.fillText(Math.round(Math.hypot(p.vx,p.vy)*28.8),W/2,H-44);g.font='bold 14px sans-serif';g.fillText('km/h  |  '+(p.veh?p.veh.n:''),W/2,H-24);
 if(bustT>0){const w=Math.min(260,W*.4);g.fillStyle='rgba(0,0,0,.55)';g.fillRect(W/2-w/2,H*.72,w,14);g.fillStyle='#ff4d5e';g.fillRect(W/2-w/2,H*.72,w*Math.min(1,bustT/3.5),14);g.fillStyle='#fff';g.font='bold 14px sans-serif';g.fillText('逮捕メーター（動いて脱出！）',W/2,H*.72-6)}
 if(PROMPT){g.font='bold 22px sans-serif';g.fillStyle='#ffe14a';g.fillText(PROMPT,W/2,H*.62)}
 if(MS){const ls=missionLines(),h=40+ls.length*20;g.shadowBlur=0;g.fillStyle='rgba(20,28,38,.78)';g.fillRect(12,208,270,h);g.textAlign='left';g.fillStyle='#7fe0ff';g.font='bold 16px sans-serif';g.fillText('◆ '+MS.name,22,230);g.fillStyle='#fff';g.font='14px sans-serif';ls.forEach((l,i)=>g.fillText(l,22,252+i*20))}
 g.restore();drawNotes()}
function miniExtras(ex){const u=1/(MM.R/950),dot=(x,y,r,fill)=>{g.fillStyle=fill;g.beginPath();g.arc(x,y,r*u,0,PI*2);g.fill();g.strokeStyle='#fff';g.lineWidth=2*u;g.stroke()};
 for(const q of GARAGES)if(Math.abs(q.x-p.x)<ex*1.2&&Math.abs(q.y-p.y)<ex*1.2)dot(q.x,q.y,7,'#33ff99');
 for(const m of MPOS)if(Math.abs(m.x-p.x)<ex*1.2&&Math.abs(m.y-p.y)<ex*1.2)dot(m.x,m.y,7,m.id=='A'?'#2ad4ff':m.id=='B'?'#ff4d5e':'#ffa63a');
 for(const c of cars)if(c.t=='blk'&&Math.abs(c.x-p.x)<ex&&Math.abs(c.y-p.y)<ex){g.fillStyle=c.mesh.l1?'#ff4d5e':'#ffa63a';g.fillRect(c.x-16,c.y-16,32,32)}
 if(HELI&&HELI.leave==0)dot(HELI.x,HELI.y,9,'#4d7bff');
 if(MS&&MS.type!='B'){const cp=MS.cps[MS.idx];g.strokeStyle='#ffe14a';g.lineWidth=5*u;g.setLineDash([14*u,10*u]);g.beginPath();g.moveTo(p.x,p.y);g.lineTo(cp.x,cp.y);g.stroke();g.setLineDash([]);dot(cp.x,cp.y,9,'#ffe14a')}}

start();state='title';
let last=performance.now();
function loop(n){const dt=Math.min((n-last)/1000,.05);last=n;if(state=='play'&&!menuOpen)update(dt);render(Math.min(dt*60,2.5));requestAnimationFrame(loop)}
requestAnimationFrame(loop);
</script></body></html>
