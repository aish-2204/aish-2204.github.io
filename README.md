<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Realistic Cat — HTML5 Canvas</title>
  <style>
    html,body{height:100%;margin:0;display:grid;place-items:center;background:#e9eef5;font-family:system-ui,Segoe UI,Roboto,Arial}
    .wrap{background:#fff;border-radius:16px;padding:18px 18px 10px;box-shadow:0 12px 40px rgba(0,0,0,.12)}
    canvas{display:block;border-radius:12px}
    .hint{font-size:12px;color:#556;margin-top:10px}
  </style>
</head>
<body>
  <div class="wrap">
    <canvas id="c" width="720" height="520"></canvas>
    <div class="hint">100% drawn with Canvas (no images). Refresh to get slightly different fur strokes.</div>
  </div>

<script>
const canvas = document.getElementById("c");
const ctx = canvas.getContext("2d");

const W = canvas.width, H = canvas.height;
const rand = (a,b)=>a+Math.random()*(b-a);

function ellipsePath(x,y,rx,ry,rot=0){
  ctx.beginPath();
  ctx.ellipse(x,y,rx,ry,rot,0,Math.PI*2);
}
function fillEllipse(x,y,rx,ry,rot=0){
  ellipsePath(x,y,rx,ry,rot);
  ctx.fill();
}
function strokeEllipse(x,y,rx,ry,rot=0){
  ellipsePath(x,y,rx,ry,rot);
  ctx.stroke();
}

function softShadow(blur=18, color="rgba(0,0,0,.18)", ox=0, oy=10){
  ctx.shadowBlur = blur; ctx.shadowColor = color; ctx.shadowOffsetX = ox; ctx.shadowOffsetY = oy;
}
function clearShadow(){ ctx.shadowBlur=0; ctx.shadowColor="transparent"; ctx.shadowOffsetX=0; ctx.shadowOffsetY=0; }

function furStrokes(areaX, areaY, areaRx, areaRy, count, baseColor, darkColor){
  // Quick “realistic-ish” fur: lots of tiny curved strokes with slight randomness.
  for(let i=0;i<count;i++){
    const t = Math.random()*Math.PI*2;
    const r = Math.sqrt(Math.random());
    const x = areaX + Math.cos(t)*areaRx*r;
    const y = areaY + Math.sin(t)*areaRy*r;

    const len = rand(6, 22);
    const ang = rand(-Math.PI, Math.PI);
    const x2 = x + Math.cos(ang)*len;
    const y2 = y + Math.sin(ang)*len*0.6;

    ctx.lineWidth = rand(0.6, 1.6);
    ctx.strokeStyle = (Math.random()<0.75) ? baseColor : darkColor;
    ctx.globalAlpha = rand(0.10, 0.24);
    ctx.beginPath();
    ctx.moveTo(x,y);
    // little curve
    ctx.quadraticCurveTo(
      (x+x2)/2 + rand(-6,6),
      (y+y2)/2 + rand(-6,6),
      x2,y2
    );
    ctx.stroke();
  }
  ctx.globalAlpha = 1;
}

function drawBackground(){
  // Soft gradient background + subtle “floor”
  const g = ctx.createLinearGradient(0,0,0,H);
  g.addColorStop(0,"#eaf2ff");
  g.addColorStop(1,"#d7e3f5");
  ctx.fillStyle = g;
  ctx.fillRect(0,0,W,H);

  // floor ellipse
  softShadow(30,"rgba(0,0,0,.18)",0,18);
  ctx.fillStyle = "rgba(0,0,0,.18)";
  fillEllipse(W*0.52, H*0.82, 240, 58);
  clearShadow();
}

function drawCat(){
  // Cat positioning
  const cx = W*0.48, cy = H*0.60;

  // Colors (tabby-ish brown/gray)
  const furBase = "#8b7a6a";
  const furMid  = "#7a6b5d";
  const furDark = "#3c332d";
  const muzzle  = "#f5f1ea";
  const innerEar= "#d8a0a3";

  // BODY
  softShadow(22,"rgba(0,0,0,.25)",0,14);
  let bodyGrad = ctx.createRadialGradient(cx-40,cy-30,20, cx,cy+10, 240);
  bodyGrad.addColorStop(0, "#9a8a7a");
  bodyGrad.addColorStop(0.55, furBase);
  bodyGrad.addColorStop(1, furMid);
  ctx.fillStyle = bodyGrad;
  fillEllipse(cx, cy, 190, 120, 0.02);
  clearShadow();

  // Chest highlight
  ctx.fillStyle = "rgba(255,255,255,.12)";
  fillEllipse(cx-60, cy+20, 90, 65, -0.2);

  // Tail (curved, fluffy)
  ctx.lineCap = "round";
  ctx.lineJoin = "round";
  ctx.beginPath();
  ctx.moveTo(cx+150, cy+10);
  ctx.bezierCurveTo(cx+260, cy-20, cx+290, cy+70, cx+220, cy+110);
  ctx.strokeStyle = furBase;
  ctx.lineWidth = 26;
  ctx.globalAlpha = 0.95;
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(cx+150, cy+10);
  ctx.bezierCurveTo(cx+260, cy-20, cx+290, cy+70, cx+220, cy+110);
  ctx.strokeStyle = furDark;
  ctx.lineWidth = 10;
  ctx.globalAlpha = 0.20;
  ctx.stroke();
  ctx.globalAlpha = 1;

  // LEGS / PAWS
  function leg(x,y,rx,ry){
    const lg = ctx.createLinearGradient(x,y-ry, x,y+ry);
    lg.addColorStop(0, furBase);
    lg.addColorStop(1, furMid);
    ctx.fillStyle = lg;
    fillEllipse(x,y,rx,ry,0.05);
    // paw tip
    ctx.fillStyle = "rgba(245,241,234,.95)";
    fillEllipse(x, y+ry*0.55, rx*0.68, ry*0.38);
  }
  leg(cx-90, cy+120, 44, 70);
  leg(cx-25, cy+128, 40, 66);
  leg(cx+55, cy+128, 44, 70);

  // HEAD
  const hx = cx-170, hy = cy-70;

  softShadow(18,"rgba(0,0,0,.22)",0,10);
  let headGrad = ctx.createRadialGradient(hx-20,hy-20,10, hx,hy, 110);
  headGrad.addColorStop(0,"#a39484");
  headGrad.addColorStop(0.6,furBase);
  headGrad.addColorStop(1,furMid);
  ctx.fillStyle = headGrad;
  fillEllipse(hx, hy, 95, 88, -0.03);
  clearShadow();

  // EARS (triangles with inner ear)
  function ear(px,py,flip=1){
    // outer ear
    ctx.fillStyle = furMid;
    ctx.beginPath();
    ctx.moveTo(px,py);
    ctx.lineTo(px+flip*40, py-62);
    ctx.lineTo(px+flip*72, py+6);
    ctx.closePath();
    ctx.fill();

    // inner ear
    ctx.fillStyle = innerEar;
    ctx.globalAlpha = 0.85;
    ctx.beginPath();
    ctx.moveTo(px+flip*12,py+2);
    ctx.lineTo(px+flip*40, py-42);
    ctx.lineTo(px+flip*58, py+6);
    ctx.closePath();
    ctx.fill();
    ctx.globalAlpha = 1;

    // ear fur edge
    ctx.strokeStyle = "rgba(60,51,45,.35)";
    ctx.lineWidth = 2;
    ctx.stroke();
  }
  ear(hx-55, hy-52, -1);
  ear(hx+55, hy-52,  1);

  // MUZZLE + NOSE
  ctx.fillStyle = muzzle;
  fillEllipse(hx-20, hy+35, 42, 32, 0.12);
  fillEllipse(hx+20, hy+35, 42, 32, -0.12);

  // nose
  ctx.fillStyle = "#b76b6e";
  ctx.beginPath();
  ctx.moveTo(hx, hy+26);
  ctx.lineTo(hx-14, hy+40);
  ctx.lineTo(hx+14, hy+40);
  ctx.closePath();
  ctx.fill();

  // mouth
  ctx.strokeStyle = "rgba(0,0,0,.55)";
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(hx, hy+40);
  ctx.quadraticCurveTo(hx-10, hy+52, hx-22, hy+46);
  ctx.moveTo(hx, hy+40);
  ctx.quadraticCurveTo(hx+10, hy+52, hx+22, hy+46);
  ctx.stroke();

  // EYES (green-ish with highlights)
  function eye(ex,ey){
    // eye socket shadow
    ctx.fillStyle = "rgba(0,0,0,.12)";
    fillEllipse(ex, ey+2, 22, 16);

    // iris gradient
    const ig = ctx.createRadialGradient(ex-6,ey-6,3, ex,ey, 22);
    ig.addColorStop(0,"#c7f5b8");
    ig.addColorStop(0.45,"#4bb36e");
    ig.addColorStop(1,"#1b5d3a");

    ctx.fillStyle = ig;
    fillEllipse(ex, ey, 18, 14);

    // pupil
    ctx.fillStyle = "rgba(0,0,0,.85)";
    fillEllipse(ex, ey, 4.6, 12.5);

    // outer eye line
    ctx.strokeStyle = "rgba(0,0,0,.45)";
    ctx.lineWidth = 2;
    strokeEllipse(ex, ey, 18, 14);

    // highlights
    ctx.fillStyle = "rgba(255,255,255,.9)";
    fillEllipse(ex-6, ey-6, 4, 4);
    ctx.fillStyle = "rgba(255,255,255,.55)";
    fillEllipse(ex-1, ey-2, 2.2, 2.2);
  }
  eye(hx-30, hy+5);
  eye(hx+30, hy+5);

  // TABBY STRIPES (head + body)
  ctx.strokeStyle = "rgba(30,25,22,.55)";
  ctx.lineCap = "round";

  // classic forehead "M"
  ctx.lineWidth = 4;
  ctx.beginPath();
  ctx.moveTo(hx-40, hy-20);
  ctx.quadraticCurveTo(hx-20, hy-45, hx, hy-25);
  ctx.quadraticCurveTo(hx+20, hy-45, hx+40, hy-20);
  ctx.stroke();

  // cheek stripes
  ctx.lineWidth = 3.5;
  for(let i=0;i<3;i++){
    const yy = hy+18+i*10;
    ctx.beginPath();
    ctx.moveTo(hx-54, yy);
    ctx.quadraticCurveTo(hx-30, yy+6, hx-12, yy+2);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(hx+54, yy);
    ctx.quadraticCurveTo(hx+30, yy+6, hx+12, yy+2);
    ctx.stroke();
  }

  // body stripes (soft curved)
  ctx.globalAlpha = 0.45;
  ctx.lineWidth = 10;
  for(let i=0;i<5;i++){
    const sx = cx-110 + i*55;
    ctx.beginPath();
    ctx.moveTo(sx, cy-10);
    ctx.quadraticCurveTo(sx+10, cy+30, sx+25, cy+60);
    ctx.stroke();
  }
  ctx.globalAlpha = 1;

  // WHISKERS (white and a few dark ones)
  function whisker(x1,y1,x2,y2,w,col,alpha=0.9){
    ctx.strokeStyle = col;
    ctx.globalAlpha = alpha;
    ctx.lineWidth = w;
    ctx.beginPath();
    ctx.moveTo(x1,y1);
    ctx.quadraticCurveTo((x1+x2)/2, (y1+y2)/2 + rand(-8,8), x2,y2);
    ctx.stroke();
    ctx.globalAlpha = 1;
  }
  const wy = hy+44;
  for(let i=0;i<5;i++){
    whisker(hx-25, wy+i*4, hx-140, wy-14+i*10, 1.4, "rgba(255,255,255,.95)", 0.9);
    whisker(hx+25, wy+i*4, hx+140, wy-14+i*10, 1.4, "rgba(255,255,255,.95)", 0.9);
  }
  // a couple darker whiskers
  whisker(hx-18, wy+8, hx-120, wy+2, 1.0, "rgba(60,51,45,.7)", 0.5);
  whisker(hx+18, wy+8, hx+120, wy+2, 1.0, "rgba(60,51,45,.7)", 0.5);

  // FUR TEXTURE (over body + head)
  // Keep it “realistic-ish” but still pure code: many light strokes.
  furStrokes(cx, cy, 175, 110, 1400, "rgba(255,255,255,.40)", "rgba(20,16,14,.35)");
  furStrokes(hx, hy, 90, 80, 650, "rgba(255,255,255,.35)", "rgba(20,16,14,.30)");

  // Outline hint (very subtle)
  ctx.strokeStyle = "rgba(0,0,0,.10)";
  ctx.lineWidth = 1.2;
  strokeEllipse(cx, cy, 190, 120, 0.02);
  strokeEllipse(hx, hy, 95, 88, -0.03);

  // Tiny text (optional)
  ctx.fillStyle = "rgba(20,40,80,.75)";
  ctx.font = "600 18px system-ui,Segoe UI,Roboto,Arial";
  ctx.fillText("Canvas Cat (no images)", 24, 34);
}

drawBackground();
drawCat();
</script>
</body>
</html>
