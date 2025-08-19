# Lakshmi-
import React, { useEffect, useMemo, useState } from "react"; import { ShoppingCart, Plus, Trash2, Edit3, X, Image as ImageIcon, Package, CreditCard, CheckCircle2, Search, Filter, Banknote, Truck, Store, Settings, Upload, ChevronDown } from "lucide-react"; import { motion, AnimatePresence } from "framer-motion";

/**

Lakshmi — Tienda online minimalista (solo front-end)


---

✔️ Catálogo con búsqueda, filtros y categorías dinámicas

✔️ Carrito, resumen, checkout con pago por transferencia bancaria

✔️ Métodos de envío (incluye Correo Argentino) y retiro/entrega

✔️ Panel de administración para agregar/editar/eliminar productos y datos de la tienda

✔️ Persistencia local (localStorage) + exportar/importar respaldo .json

✔️ Estética clara y minimalista con Tailwind + animaciones sutiles

Nota: Es una app 100% del lado del cliente para comenzar rápido.

Para producción real, conviene agregar backend/pasarela de pago. */


// Utilidades const currency = (n) => (n ?? 0).toLocaleString("es-AR", { style: "currency", currency: "ARS" }); const uid = () => Math.random().toString(36).slice(2, 10);

// Claves de almacenamiento local const LS_PRODUCTS = "lakshmi_products_v1"; const LS_SETTINGS = "lakshmi_settings_v1"; const LS_LOGO = "lakshmi_logo_dataurl_v1";

// Datos por defecto const defaultSettings = { storeName: "Lakshmi", tagline: "Accesorios de acero quirúrgico", bank: { alias: "LAKSHMI.ACCESORIOS", cbu: "0000000000000000000000", titular: "Nombre Apellido", banco: "Banco X", instrucciones: "Transferir el total y enviar comprobante por WhatsApp/Instagram." }, shipping: { methods: [ { id: "correo", name: "Correo Argentino", desc: "A todo el país. Costo según destino/peso." }, { id: "mensajeria", name: "Mensajería/Envío local", desc: "Envíos en la ciudad (coordinar vía chat)." }, { id: "retiro", name: "Retiro en persona", desc: "Gratis. Coordinar punto y horario." } ], defaultMethodId: "correo" }, contact: { instagram: "https://instagram.com/tuusuario", whatsapp: "https://wa.me/5490000000000" } };

const demoProducts = [ { id: uid(), name: "Aro huggie acero 316L", description: "Aro huggie minimalista hipoalergénico.", price: 6500, category: "Aros", stock: 15, imageUrl: "https://images.unsplash.com/photo-1515562141207-7a88fb7ce338?q=80&w=1200&auto=format&fit=crop" }, { id: uid(), name: "Collar dije luna", description: "Cadena fina con dije de luna en acero quirúrgico.", price: 9800, category: "Collares", stock: 10, imageUrl: "https://images.unsplash.com/photo-1603575449153-2876b8f90b21?q=80&w=1200&auto=format&fit=crop" }, { id: uid(), name: "Pulsera eslabones", description: "Pulsera ajustable estilo clásico.", price: 7200, category: "Pulseras", stock: 8, imageUrl: "https://images.unsplash.com/photo-1603570419965-6012372a0d13?q=80&w=1200&auto=format&fit=crop" } ];

// Componente principal export default function LakshmiStore() { // Estado de catálogo y ajustes const [products, setProducts] = useState(() => { const saved = localStorage.getItem(LS_PRODUCTS); return saved ? JSON.parse(saved) : demoProducts; }); const [settings, setSettings] = useState(() => { const saved = localStorage.getItem(LS_SETTINGS); return saved ? JSON.parse(saved) : defaultSettings; }); const [logoDataUrl, setLogoDataUrl] = useState(() => localStorage.getItem(LS_LOGO) || "");

// Estado UI const [query, setQuery] = useState(""); const [category, setCategory] = useState("Todos"); const [cart, setCart] = useState([]); // {id, qty} const [showCart, setShowCart] = useState(false); const [admin, setAdmin] = useState(false); const [editing, setEditing] = useState(null); // product or null const [checkout, setCheckout] = useState(false); const [orderPlaced, setOrderPlaced] = useState(null); // order object const [shippingMethod, setShippingMethod] = useState(settings.shipping.defaultMethodId);

// Persistencia useEffect(() => localStorage.setItem(LS_PRODUCTS, JSON.stringify(products)), [products]); useEffect(() => localStorage.setItem(LS_SETTINGS, JSON.stringify(settings)), [settings]); useEffect(() => { if (logoDataUrl) localStorage.setItem(LS_LOGO, logoDataUrl); }, [logoDataUrl]);

// Derivados const categories = useMemo(() => ["Todos", ...Array.from(new Set(products.map(p => p.category))).sort()], [products]); const filtered = useMemo(() => { const q = query.trim().toLowerCase(); return products.filter(p => { const okCat = category === "Todos" || p.category === category; const okQ = !q || [p.name, p.description, p.category].some(v => v?.toLowerCase().includes(q)); return okCat && okQ; }); }, [products, query, category]);

const cartItems = useMemo(() => cart.map(({ id, qty }) => ({ ...products.find(p => p.id === id), qty })), [cart, products]); const subtotal = useMemo(() => cartItems.reduce((s, i) => s + (i.price * i.qty), 0), [cartItems]);

// Acciones carrito const addToCart = (id) => setCart(prev => { const e = prev.find(i => i.id === id); if (e) return prev.map(i => i.id === id ? { ...i, qty: Math.min(i.qty + 1, (products.find(p => p.id === id)?.stock ?? 1)) } : i); return [...prev, { id, qty: 1 }]; }); const changeQty = (id, qty) => setCart(prev => prev.map(i => i.id === id ? { ...i, qty: Math.max(1, Math.min(qty, (products.find(p => p.id === id)?.stock ?? 1))) } : i)); const removeFromCart = (id) => setCart(prev => prev.filter(i => i.id !== id)); const clearCart = () => setCart([]);

// CRUD Productos const upsertProduct = (p) => { setProducts(prev => { const idx = prev.findIndex(x => x.id === p.id); if (idx === -1) return [p, ...prev]; const cp = [...prev]; cp[idx] = p; return cp; }); setEditing(null); }; const deleteProduct = (id) => setProducts(prev => prev.filter(p => p.id !== id));

// Logo upload const onLogoChange = (file) => { if (!file) return; const reader = new FileReader(); reader.onload = () => setLogoDataUrl(reader.result); reader.readAsDataURL(file); };

// Checkout const placeOrder = (cliente) => { const order = { id: LK-${Date.now().toString().slice(-6)}, date: new Date().toISOString(), items: cartItems, subtotal, shippingMethod, cliente }; setOrderPlaced(order); clearCart(); setCheckout(false); setShowCart(false); };

return ( <div className="min-h-screen bg-white text-gray-900"> <Header settings={settings} logoDataUrl={logoDataUrl} onLogoChange={onLogoChange} cartCount={cart.reduce((s, i) => s + i.qty, 0)} onToggleCart={() => setShowCart(v => !v)} admin={admin} setAdmin={setAdmin} setCheckout={setCheckout} />

{/* Barra de búsqueda y filtros */}
  <div className="max-w-6xl mx-auto px-4 md:px-6 lg:px-8 py-6">
    <div className="flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
      <div className="flex items-center gap-2 w-full md:w-1/2 border rounded-2xl px-3 py-2">
        <Search className="w-4 h-4" />
        <input value={query} onChange={e => setQuery(e.target.value)} placeholder="Buscar productos..." className="w-full outline-none bg-transparent" />
      </div>
      <div className="flex items-center gap-2">
        <Filter className="w-4 h-4" />
        <select value={category} onChange={(e) => setCategory(e.target.value)} className="border rounded-2xl px-3 py-2">
          {categories.map(c => <option key={c}>{c}</option>)}
        </select>
      </div>
    </div>
  </div>

  {/* Grid de productos */}
  <main className="max-w-6xl mx-auto px-4 md:px-6 lg:px-8 pb-24">
    {filtered.length === 0 ? (
      <EmptyState onAdd={() => setAdmin(true)} />
    ) : (
      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
        {filtered.map(p => (
          <motion.div key={p.id} layout initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0 }}>
            <ProductCard
              product={p}
              onAdd={() => addToCart(p.id)}
              onEdit={() => { setEditing(p); setAdmin(true); }}
              onDelete={() => deleteProduct(p.id)}
              admin={admin}
            />
          </motion.div>
        ))}
      </div>
    )}
  </main>

  {/* Panel admin */}
  <AnimatePresence>
    {admin && (
      <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0, y: 20 }} className="fixed bottom-0 left-0 right-0 border-t bg-white/80 backdrop-blur supports-[backdrop-filter]:bg-white/60">
        <div className="max-w-6xl mx-auto px-4 md:px-6 lg:px-8 py-4">
          <div className="flex items-center justify-between">
            <div className="flex items-center gap-2">
              <Settings className="w-5 h-5" />
              <span className="font-medium">Panel de administración</span>
            </div>
            <button className="text-sm underline" onClick={() => setAdmin(false)}>Cerrar</button>
          </div>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mt-4">
            <StoreSettings settings={settings} setSettings={setSettings} />
            <ProductEditor key={editing?.id || "new"} editing={editing} onSave={upsertProduct} onCancel={() => setEditing(null)} />
            <DataBackup products={products} settings={settings} setProducts={setProducts} setSettings={setSettings} setLogoDataUrl={setLogoDataUrl} />
          </div>
        </div>
      </motion.div>
    )}
  </AnimatePresence>

  {/* Carrito lateral */}
  <AnimatePresence>
    {showCart && (
      <motion.aside initial={{ x: 400 }} animate={{ x: 0 }} exit={{ x: 400 }} transition={{ type: "spring", stiffness: 260, damping: 30 }} className="fixed top-0 right-0 w-full sm:w-[420px] h-full bg-white shadow-2xl border-l z-50">
        <div className="flex items-center justify-between px-4 py-3 border-b">
          <div className="flex items-center gap-2"><ShoppingCart className="w-5 h-5" /><span className="font-medium">Carrito</span></div>
          <button onClick={() => setShowCart(false)} className="p-2 rounded-xl hover:bg-gray-100"><X className="w-5 h-5" /></button>
        </div>
        <div className="p-4 flex flex-col h-full">
          <div className="flex-1 space-y-3 overflow-auto pr-1">
            {cartItems.length === 0 ? (
              <div className="text-sm text-gray-500">Tu carrito está vacío.</div>
            ) : cartItems.map(i => (
              <div key={i.id} className="flex gap-3 items-center border rounded-2xl p-2">
                <img src={i.imageUrl} className="w-16 h-16 object-cover rounded-xl"/>
                <div className="flex-1">
                  <div className="text-sm font-medium">{i.name}</div>
                  <div className="text-xs text-gray-500">{currency(i.price)} • Stock: {i.stock}</div>
                  <div className="flex items-center gap-2 mt-1">
                    <input type="number" className="w-16 border rounded-xl px-2 py-1" value={i.qty} min={1} max={i.stock} onChange={(e) => changeQty(i.id, Number(e.target.value))} />
                    <button className="px-2 py-1 text-xs border rounded-xl" onClick={() => removeFromCart(i.id)}><Trash2 className="w-3 h-3 inline"/> Quitar</button>
                  </div>
                </div>
                <div className="text-sm font-medium">{currency(i.price * i.qty)}</div>
              </div>
            ))}
          </div>
          <div className="border-t pt-3 space-y-2">
            <div className="flex items-center justify-between text-sm"><span>Subtotal</span><span>{currency(subtotal)}</span></div>
            <div className="text-xs text-gray-500">El costo de envío se coordina según método. Selecciónalo en el checkout.</div>
            <div className="flex gap-2 pt-2">
              <button className="flex-1 rounded-2xl px-4 py-2 border" onClick={clearCart}>Vaciar</button>
              <button disabled={cartItems.length===0} className="flex-1 rounded-2xl px-4 py-2 bg-gray-900 text-white disabled:opacity-50" onClick={() => { setCheckout(true); }}>
                Ir a pagar
              </button>
            </div>
          </div>
        </div>
      </motion.aside>
    )}
  </AnimatePresence>

  {/* Checkout modal */}
  <AnimatePresence>
    {checkout && (
      <CheckoutModal
        onClose={() => setCheckout(false)}
        settings={settings}
        cartItems={cartItems}
        subtotal={subtotal}
        shippingMethod={shippingMethod}
        setShippingMethod={setShippingMethod}
        onPlaceOrder={placeOrder}
      />
    )}
  </AnimatePresence>

  {/* Orden colocada */}
  <AnimatePresence>
    {orderPlaced && (
      <OrderPlaced order={orderPlaced} settings={settings} onClose={() => setOrderPlaced(null)} />
    )}
  </AnimatePresence>

  <Footer settings={settings} />
</div>

); }

function Header({ settings, logoDataUrl, onLogoChange, cartCount, onToggleCart, admin, setAdmin, setCheckout }) { return ( <header className="border-b bg-white/70 backdrop-blur supports-[backdrop-filter]:bg-white/60 sticky top-0 z-40"> <div className="max-w-6xl mx-auto px-4 md:px-6 lg:px-8 py-3 flex items-center justify-between"> <div className="flex items-center gap-3"> <div className="w-10 h-10 rounded-2xl bg-gray-100 flex items-center justify-center overflow-hidden"> {logoDataUrl ? <img src={logoDataUrl} alt="logo" className="w-full h-full object-cover"/> : <ImageIcon className="w-5 h-5" />} </div> <div> <div className="font-semibold text-lg leading-tight">{settings.storeName}</div> <div className="text-xs text-gray-500">{settings.tagline}</div> </div> </div> <div className="flex items-center gap-2"> <button onClick={() => setAdmin(v => !v)} className={hidden sm:flex items-center gap-2 px-3 py-2 rounded-2xl border ${admin ? 'bg-gray-900 text-white' : ''}}><Settings className="w-4 h-4"/> Admin</button> <button onClick={() => setCheckout(true)} className="hidden sm:flex items-center gap-2 px-3 py-2 rounded-2xl border"><CreditCard className="w-4 h-4"/> Pagar</button> <button onClick={onToggleCart} className="relative p-2 rounded-xl border hover:bg-gray-50"> <ShoppingCart className="w-5 h-5" /> {cartCount > 0 && ( <span className="absolute -top-1 -right-1 text-[10px] px-1.5 py-0.5 rounded-full bg-gray-900 text-white">{cartCount}</span> )} </button> </div> </div> {/* Cargar logo (solo admin) /} {admin && ( <div className="max-w-6xl mx-auto px-4 md:px-6 lg:px-8 pb-3"> <label className="inline-flex items-center gap-2 text-xs cursor-pointer"> <Upload className="w-3.5 h-3.5"/> <span>Subir logo (PNG/JPG)</span> <input type="file" accept="image/" className="hidden" onChange={(e) => onLogoChange(e.target.files?.[0])} /> </label> </div> )} </header> ); }

function ProductCard({ product, onAdd, onEdit, onDelete, admin }) { return ( <div className="border rounded-3xl overflow-hidden bg-white group shadow-sm hover:shadow-md transition-shadow"> <div className="aspect-[4/3] bg-gray-50 overflow-hidden"> <img src={product.imageUrl} alt={product.name} className="w-full h-full object-cover group-hover:scale-[1.02] transition-transform"/> </div> <div className="p-4"> <div className="flex items-start justify-between gap-2"> <div> <h3 className="font-medium leading-snug">{product.name}</h3> <p className="text-xs text-gray-500">{product.category}</p> </div> <div className="text-sm font-semibold whitespace-nowrap">{currency(product.price)}</div> </div> <p className="text-sm text-gray-600 mt-2 line-clamp-2 min-h-[40px]">{product.description}</p> <div className="flex items-center justify-between mt-3"> <button onClick={onAdd} className="flex items-center gap-2 px-3 py-2 rounded-2xl border"><Plus className="w-4 h-4"/> Agregar</button> <div className="text-xs text-gray-500">Stock: {product.stock}</div> </div> {admin && ( <div className="flex items-center gap-2 mt-3"> <button onClick={onEdit} className="text-xs px-3 py-1.5 rounded-xl border"><Edit3 className="w-3.5 h-3.5 inline"/> Editar</button> <button onClick={onDelete} className="text-xs px-3 py-1.5 rounded-xl border text-red-600"><Trash2 className="w-3.5 h-3.5 inline"/> Eliminar</button> </div> )} </div> </div> ); }

function EmptyState({ onAdd }) { return ( <div className="border rounded-3xl p-10 text-center bg-gray-50"> <Package className="w-8 h-8 mx-auto"/> <h3 className="font-medium mt-2">Aún no hay productos</h3> <p className="text-sm text-gray-600 mt-1">Agregá los primeros desde el panel de administración.</p> <button onClick={onAdd} className="mt-4 px-4 py-2 rounded-2xl border">Abrir admin</button> </div> ); }

function StoreSettings({ settings, setSettings }) { const [open, setOpen] = useState(true); return ( <div className="border rounded-3xl p-4 bg-white"> <button className="w-full flex items-center justify-between" onClick={() => setOpen(v=>!v)}> <div className="flex items-center gap-2 font-medium"><Settings className="w-4 h-4"/> Datos de la tienda</div> <ChevronDown className={w-4 h-4 transition-transform ${open ? 'rotate-180' : ''}}/> </button> {open && ( <div className="space-y-3 mt-4"> <TextInput label="Nombre" value={settings.storeName} onChange={(v)=>setSettings(s=>({...s, storeName:v}))} /> <TextInput label="Descripción corta" value={settings.tagline} onChange={(v)=>setSettings(s=>({...s, tagline:v}))} /> <div className="pt-2"> <h4 className="text-sm font-medium mb-2">Pago por transferencia bancaria</h4> <TextInput label="Alias" value={settings.bank.alias} onChange={(v)=>setSettings(s=>({...s, bank:{...s.bank, alias:v}}))} /> <TextInput label="CBU" value={settings.bank.cbu} onChange={(v)=>setSettings(s=>({...s, bank:{...s.bank, cbu:v}}))} /> <TextInput label="Titular" value={settings.bank.titular} onChange={(v)=>setSettings(s=>({...s, bank:{...s.bank, titular:v}}))} /> <TextInput label="Banco" value={settings.bank.banco} onChange={(v)=>setSettings(s=>({...s, bank:{...s.bank, banco:v}}))} /> <TextArea label="Instrucciones" value={settings.bank.instrucciones} onChange={(v)=>setSettings(s=>({...s, bank:{...s.bank, instrucciones:v}}))} /> </div> <div className="pt-2"> <h4 className="text-sm font-medium mb-2">Envíos</h4> {settings.shipping.methods.map((m, idx) => ( <div key={m.id} className="border rounded-2xl p-3 mb-2"> <div className="flex gap-2"> <TextInput label="Nombre" value={m.name} onChange={(v)=>updateMethod(idx, { ...m, name:v })} /> <TextInput label="ID" value={m.id} onChange={(v)=>updateMethod(idx, { ...m, id:v })} /> </div> <TextInput label="Descripción" value={m.desc} onChange={(v)=>updateMethod(idx, { ...m, desc:v })} /> <div className="flex items-center gap-2 text-xs mt-2"> <input type="radio" checked={settings.shipping.defaultMethodId === m.id} onChange={()=>setSettings(s=>({...s, shipping:{...s.shipping, defaultMethodId:m.id}}))} /> <span>Método por defecto</span> </div> <div className="mt-2 text-right"> <button className="text-xs underline" onClick={()=>removeMethod(idx)}>Quitar</button> </div> </div> ))} <button className="mt-1 text-sm px-3 py-1.5 rounded-xl border" onClick={()=>addMethod()}>Agregar método</button> </div> <div className="pt-2"> <h4 className="text-sm font-medium mb-2">Contacto</h4> <TextInput label="Instagram (URL)" value={settings.contact.instagram} onChange={(v)=>setSettings(s=>({...s, contact:{...s.contact, instagram:v}}))} /> <TextInput label="WhatsApp (URL)" value={settings.contact.whatsapp} onChange={(v)=>setSettings(s=>({...s, contact:{...s.contact, whatsapp:v}}))} /> </div> </div> )} </div> );

function updateMethod(idx, m) { setSettings(s=>{ const methods=[...s.shipping.methods]; methods[idx]=m; return { ...s, shipping:{...s.shipping, methods} }; }); } function removeMethod(idx){ setSettings(s=>{ const methods=s.shipping.methods.filter((_,i)=>i!==idx); const defaultId = methods[0]?.id || ""; return { ...s, shipping:{ methods, defaultMethodId: defaultId } }; }); } function addMethod(){ setSettings(s=>({ ...s, shipping: { ...s.shipping, methods: [...s.shipping.methods, { id: uid(), name: "Nuevo método", desc: "Descripción" }], defaultMethodId: s.shipping.defaultMethodId || s.shipping.methods[0]?.id } })); } }

function ProductEditor({ editing, onSave, onCancel }) { const [form, setForm] = useState( editing || { id: uid(), name: "", description: "", price: 0, category: "", stock: 1, imageUrl: "" } ); const onFile = (file) => { if (!file) return; const reade
