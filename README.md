# Swadpoint-
Food delivery app for swadpoint 
<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>SwadPoint</title>
<link rel="manifest" href="manifest.json"/>
<link rel="stylesheet" href="styles.css"/>
</head>
<body>
<header class="header">
  <div class="logo"><img src="logo.svg" alt="logo" style="width:48px;height:48px"/></div>
  <h1 id="appName">SwadPoint</h1>
</header>

<main class="container">
  <div id="categories" class="categories"></div>
  <div id="grid" class="grid"></div>
  <div class="footer-space"></div>

  <!-- Admin Panel -->
  <div id="adminPanel" class="admin">
    <h2>⚙️ Admin Panel</h2>
    <p class="small">Yahan se bina code ke aap items update kar sakte hain (name, price, photo).</p>

    <label>Item select:</label>
    <select id="foodSelect" onchange="loadItem()"></select><br/><br/>

    <label>Naya Naam:</label><br/>
    <input id="newName" type="text" class="input"/><br/><br/>

    <label>Nayi Price (₹):</label><br/>
    <input id="newPrice" type="number" class="input"/><br/><br/>

    <label>Photo file name (example: samosa.jpg):</label><br/>
    <input id="newImg" type="text" class="input"/><br/><br/>

    <button class="btn" onclick="updateItem()">Update Item</button>
    <button class="btn" onclick="addNewItem()">Add New Item</button>
    <button class="btn" onclick="downloadZip()">Download ZIP</button>
    <p class="small">Note: Images ko folder me rakhna (same folder) ya external URL de sakte ho.</p>
  </div>
</main>

<button id="cartBtn" class="cart-bar">
  <div class="badge" id="cartCount">0</div>
  <div>Cart</div>
</button>

<script>const DELIVERY_PER_KM = 15; const WHATSAPP_NUMBER = '917091084967';</script>
<script src="app.js"></script>
<script>
if('serviceWorker' in navigator){ navigator.serviceWorker.register('sw.js').catch(()=>console.log('sw fail')) }
</script>
</body>
</html>
:root{--bg:#111;--card:#1a1a1a;--accent:#d40000;--muted:#bfbfbf;}
*{box-sizing:border-box}
body{margin:0;font-family:Arial,Helvetica,sans-serif;background:var(--bg);color:#fff}
.header{background:var(--accent);padding:12px 14px;display:flex;align-items:center;gap:12px}
.logo{width:48px;height:48px;border-radius:8px;overflow:hidden;background:#000;display:flex;align-items:center;justify-content:center}
.header h1{margin:0;font-size:20px}
.container{padding:12px}
.categories{display:flex;gap:8px;overflow-x:auto;padding-bottom:8px;margin-bottom:8px}
.cat{background:transparent;border:1px solid #2a2a2a;padding:8px 12px;border-radius:20px;cursor:pointer;color:var(--muted)}
.cat.active{background:var(--accent);color:#fff;border-color:var(--accent)}
.grid{display:grid;grid-template-columns:repeat(1,1fr);gap:12px}
@media(min-width:600px){.grid{grid-template-columns:repeat(2,1fr)}}
.card{background:var(--card);padding:12px;border-radius:12px;display:flex;gap:12px;align-items:center}
.card img{width:120px;height:90px;object-fit:cover;border-radius:8px}
.item-info{flex:1}
.item-title{font-size:18px;margin:0 0 6px 0}
.item-meta{color:var(--muted);font-size:14px;margin-bottom:8px}
.btn{background:var(--accent);color:#fff;padding:8px 12px;border-radius:8px;text-decoration:none;display:inline-block;border:none;cursor:pointer}
.cart-bar{position:fixed;right:12px;bottom:12px;background:var(--accent);padding:12px 16px;border-radius:999px;display:flex;align-items:center;gap:12px;box-shadow:0 6px 20px rgba(0,0,0,0.4)}
.badge{background:#000;padding:6px 8px;border-radius:8px}
.cart-panel{position:fixed;left:0;right:0;bottom:0;background:#0f0f0f;border-top-left-radius:16px;border-top-right-radius:16px;padding:12px;max-height:60vh;overflow:auto}
.input{width:100%;padding:8px;border-radius:8px;border:1px solid #2a2a2a;background:#0b0b0b;color:#fff}
.small{font-size:13px;color:var(--muted)}
.admin{padding:12px;background:#0b0b0b;border-radius:12px;margin-top:16px}
.footer-space{height:72px}
// SwadPoint app logic with admin panel (editable)
const MENU = [
  {id:'snacks', name:'Snacks', items:[
    {id:'samosa', title:'Samosa', price:20, img:'samosa.jpg'},
    {id:'litti', title:'Litti', price:30, img:'litti.jpg'},
    {id:'pakode', title:'Pakaude', price:25, img:'pakaude.jpg'},
    {id:'aaluchap', title:'Aalu Chap', price:35, img:'aaluchap.jpg'}
  ]},
  {id:'sweets', name:'Sweets', items:[
    {id:'kachri', title:'Kachri', price:40, img:'kachri.jpg'},
    {id:'gaja', title:'Gaja', price:45, img:'gaja.jpg'},
    {id:'khaja', title:'Khaja', price:50, img:'khaja.jpg'},
    {id:'balushahi', title:'Balushahi', price:30, img:'balushahi.jpg'},
    {id:'rasgulla', title:'Rasgulla', price:60, img:'rasgulla.jpg'},
    {id:'gulabjamun', title:'Gulabjamun', price:40, img:'gulabjamun.jpg'},
    {id:'buniya', title:'Buniya Jalebi', price:70, img:'buniya.jpg'}
  ]},
  {id:'drinks', name:'Drinks', items:[
    {id:'chai', title:'Chai', price:15, img:'chai.jpg'}
  ]}
];

const DELIVERY_PER_KM = 15; // ₹15 per km
const WHATSAPP_NUMBER = '917091084967';

const CART_KEY = 'swad_cart_v1';

function qs(sel){return document.querySelector(sel)}
function qsa(sel){return Array.from(document.querySelectorAll(sel))}

function init(){
  renderCategories();
  renderItems();
  populateAdminSelect();
  renderCartCount();
  qs('#cartBtn').onclick = openCartPanel;
}

function renderCategories(){
  const catWrap = qs('#categories');
  catWrap.innerHTML = '';
  MENU.forEach((cat, idx) => {
    const b = document.createElement('button');
    b.className = 'cat' + (idx===0 ? ' active' : '');
    b.textContent = cat.name;
    b.onclick = () => { qsa('.cat').forEach(x=>x.classList.remove('active')); b.classList.add('active'); renderItems(cat.id); }
    catWrap.appendChild(b);
  });
}

function renderItems(filterId){
  const grid = qs('#grid');
  grid.innerHTML = '';
  MENU.forEach(cat=>{
    if(filterId && cat.id !== filterId) return;
    cat.items.forEach(it=>{
      const card = document.createElement('div'); card.className='card';
      card.innerHTML = `
        <img src="${it.img}" alt="${it.title}">
        <div class="item-info">
          <div class="item-title">${it.title}</div>
          <div class="item-meta">₹ ${it.price} • ${cat.name}</div>
          <div><button class="btn" onclick="addToCart('${it.id}')">Add to Cart</button></div>
        </div>
      `;
      grid.appendChild(card);
    });
  });
}

function findItemById(id){
  for(const cat of MENU) for(const it of cat.items) if(it.id===id) return it;
  return null;
}

function addToCart(id){
  const item = findItemById(id);
  if(!item) return alert('Item not found');
  const cart = getCart(); cart.push(item); saveCart(cart); showToast(item.title + ' added to cart');
}

function getCart(){ try{ return JSON.parse(localStorage.getItem(CART_KEY)||'[]'); }catch(e){ return [] } }
function saveCart(cart){ localStorage.setItem(CART_KEY, JSON.stringify(cart)); renderCartCount(); }
function renderCartCount(){ qs('#cartCount').textContent = getCart().length; }

function openCartPanel(){
  const cart = getCart();
  let html = '<div style="display:flex;justify-content:space-between;align-items:center"><h3>Cart</h3><button onclick="closeCartPanel()" class="btn">Close</button></div>';
  if(cart.length===0) html += '<p class="small">Cart is empty</p>';
  else{
    let total=0; html += '<div>';
    cart.forEach((c, idx)=>{ total+=c.price; html += `<div style="display:flex;justify-content:space-between;margin:8px 0"><div>${c.title}</div><div>₹${c.price}</div></div>`; });
    html += '</div>';
    html += `<hr><div style="display:flex;gap:8px;margin:8px 0"><input id="distance" class="input" placeholder="Delivery distance (km)" type="number"/><input id="address" class="input" placeholder="Address (for WhatsApp)" /></div>`;
    html += `<div style="margin-top:8px"><b>Items Total:</b> ₹${total}</div>`;
    html += `<div style="margin-top:8px"><b>Delivery:</b> ₹<span id="deliveryFee">0</span> (₹${DELIVERY_PER_KM}/km)</div>`;
    html += `<div style="margin-top:8px"><b>Grand Total:</b> ₹<span id="grandTotal">${total}</span></div>`;
    html += `<div style="margin-top:12px"><button class="btn" onclick="checkout()">Checkout via WhatsApp</button> <button class="btn" style="background:#333" onclick="clearCart()">Clear Cart</button></div>`;
  }
  const panel = document.createElement('div'); panel.className='cart-panel'; panel.id='cartPanel'; panel.innerHTML = html;
  document.body.appendChild(panel);
  const distInput = qs('#distance'); if(distInput) distInput.oninput = updateDelivery;
}

function closeCartPanel(){ const el = qs('#cartPanel'); if(el) el.remove(); }
function clearCart(){ localStorage.removeItem(CART_KEY); renderCartCount(); closeCartPanel(); showToast('Cart cleared'); }

function updateDelivery(){
  const d = parseFloat(qs('#distance').value || '0');
  const cart = getCart();
  const itemsTotal = cart.reduce((s,i)=>s+i.price,0);
  const delivery = Math.max(0, Math.ceil(d) * DELIVERY_PER_KM);
  qs('#deliveryFee').textContent = delivery;
  qs('#grandTotal').textContent = itemsTotal + delivery;
}

function checkout(){
  const cart = getCart();
  if(cart.length===0) return alert('Cart empty');
  const distance = parseFloat(qs('#distance').value || '0');
  const address = encodeURIComponent(qs('#address').value || 'N/A');
  const delivery = Math.max(0, Math.ceil(distance) * DELIVERY_PER_KM);
  const itemsTotal = cart.reduce((s,i)=>s+i.price,0);
  const grand = itemsTotal + delivery;
  let msg = `Namaste SwadPoint!%0AOrder%20Details:%0A`;
  cart.forEach((c,idx)=> msg += `${idx+1}. ${c.title} - ₹${c.price}%0A`);
  msg += `%0AItems%20Total:%20₹${itemsTotal}%0ADelivery:%20₹${delivery}%0AGrand%20Total:%20₹${grand}%0AAddress:%20${address}%0AThanks!`;
  const waLink = `https://wa.me/${WHATSAPP_NUMBER}?text=${msg}`;
  localStorage.removeItem(CART_KEY); renderCartCount();
  window.open(waLink,'_blank');
  closeCartPanel();
}

function showToast(text){
  const t = document.createElement('div'); t.style.position='fixed'; t.style.bottom='90px'; t.style.left='50%'; t.style.transform='translateX(-50%)'; t.style.background='#222'; t.style.padding='10px 14px'; t.style.borderRadius='10px'; t.style.boxShadow='0 6px 18px rgba(0,0,0,0.4)'; t.style.zIndex=9999; t.textContent = text;
  document.body.appendChild(t);
  setTimeout(()=>t.remove(),1800);
}

/* ---------------- Admin Panel functions ---------------- */

function populateAdminSelect(){
  const sel = qs('#foodSelect');
  sel.innerHTML = '';
  MENU.forEach(cat=>{
    cat.items.forEach(it=>{
      const opt = document.createElement('option');
      opt.value = it.id;
      opt.textContent = `${it.title} (${cat.name})`;
      sel.appendChild(opt);
    });
  });
  loadItem();
}

function loadItem(){
  const id = qs('#foodSelect').value;
  const it = findItemById(id);
  if(!it) return;
  qs('#newName').value = it.title;
  qs('#newPrice').value = it.price;
  qs('#newImg').value = it.img;
}

function updateItem(){
  const id = qs('#foodSelect').value;
  const it = findItemById(id);
  if(!it) return alert('Item not found');
  const n = qs('#newName').value.trim();
  const p = parseFloat(qs('#newPrice').value || '0');
  const img = qs('#newImg').value.trim() || it.img;
  if(n) it.title = n;
  if(!isNaN(p) && p>0) it.price = p;
  it.img = img;
  renderItems();
  populateAdminSelect();
  showToast('Item updated');
}

function addNewItem(){
  const name = prompt('Naya item ka naam likho (example: Samosa)');
  if(!name) return;
  const price = parseFloat(prompt('Price (₹) likho', '25') || '25');
  const img = prompt('Photo filename ya URL (example: samosa.jpg)', 'samosa.jpg') || 'samosa.jpg';
  // add to first category (snacks) by default
  MENU[0].items.push({id: name.toLowerCase().replace(/\\s+/g,'_')+Date.now(), title: name, price: price, img: img});
  renderItems();
  populateAdminSelect();
  showToast('New item added');
}

/* Download simple ZIP - generates a blob with basic files (works in browser) */
function downloadZip(){
  try{
    const files = {
      'index.html': document.documentElement.outerHTML,
      'styles.css': qs('link[rel="stylesheet"]') ? fetch(qs('link[rel="stylesheet"]').getAttribute('href')).then(r=>r.text()) : Promise.resolve(''),
      'app.js': fetch('app.js').then(r=>r.text()),
      'manifest.json': JSON.stringify({name:'SwadPoint',short_name:'SwadPoint',start_url:'index.html',display:'standalone',background_color:'#111',theme_color:'#d40000',icons:[]}, null, 2),
      'logo.svg': '<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"512\" height=\"512\" viewBox=\"0 0 512 512\"><rect width=\"100%\" height=\"100%\" fill=\"#111\"/><g transform=\"translate(50,80)\"><path d=\"M80 0 L160 0 L200 80 L40 80 Z\" fill=\"#d40000\"/><circle cx=\"110\" cy=\"40\" r=\"6\" fill=\"#111\"/><text x=\"0\" y=\"170\" font-family=\"Arial\" font-size=\"48\" fill=\"#fff\" font-weight=\"700\">SwadPoint</text></g></svg>'
    };
    // read async files then create zip
    Promise.all([files['styles.css'], files['app.js']]).then(vals=>{
      const css = vals[0] || '';
      const js = vals[1] || '';
      const JSZipScript = `
      `;
      // Use client-side zip via JSZip library if available — fallback: open a new window with instructions
      try{
        if(window.JSZip){
          const zip = new JSZip();
          zip.file('index.html', files['index.html']);
          zip.file('styles.css', css);
          zip.file('app.js', js);
          zip.file('manifest.json', files['manifest.json']);
          zip.file('logo.svg', files['logo.svg']);
          zip.generateAsync({type:'blob'}).then(function(content){
            const a = document.createElement('a'); a.href = URL.createObjectURL(content); a.download = 'swadpoint.zip'; a.click();
          });
        } else {
          alert('ZIP download requires JSZip library. Upload folder manually or use Netlify (easy).');
        }
      }catch(e){ alert('ZIP error: '+e.message) }
    });
  }catch(e){ alert('Download not supported in this browser'); }
}

window.addEventListener('load', init);
self.addEventListener('install', e => {
  e.waitUntil(caches.open('swad-cache-v1').then(cache=>cache.addAll(['index.html','styles.css','app.js','manifest.json','logo.svg'])));
  self.skipWaiting();
});
self.addEventListener('activate', e => { self.clients.claim(); });
self.addEventListener('fetch', e => { e.respondWith(caches.match(e.request).then(r=>r||fetch(e.request))); });
{
  "name": "SwadPoint",
  "short_name": "SwadPoint",
  "start_url": "index.html",
  "display": "standalone",
  "background_color": "#111111",
  "theme_color": "#d40000",
  "icons": []
}
<svg xmlns="http://www.w3.org/2000/svg" width="512" height="512" viewBox="0 0 512 512">
<rect width="100%" height="100%" fill="#111"/>
<g transform="translate(50,80)">
  <path d="M80 0 L160 0 L200 80 L40 80 Z" fill="#d40000"/>
  <circle cx="110" cy="40" r="6" fill="#111"/>
  <text x="0" y="170" font-family="Arial" font-size="48" fill="#fff" font-weight="700">SwadPoint</text>
</g>
</svg>
