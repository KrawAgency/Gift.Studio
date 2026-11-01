# Gift Studio · guía para Claude

App de un solo archivo para ofrecer regalos a huéspedes de Dreams Riviera Cancún al invitarlos a la presentación de UVC. Se usa cara a cara, sobre todo en Galaxy Tab S9 Ultra (horizontal) y en celular. Se publica en GitHub Pages desde `main`: https://krawagency.github.io/Gift.Studio/

## Cómo trabajar aquí
- **Cambios quirúrgicos.** Nunca reescribir `index.html` completo. Buscar con grep por nombre de función o constante y leer solo esas líneas.
- **No leer el archivo entero.** Son ~330 KB / ~4,000 líneas. Los números de línea cambian seguido: busca por nombre, no por número.
- **Comentarios en español** con versión o fecha (`// Nov 2026 · ...`, `// v29.21 · ...`), igual que el resto del código.
- **Antes de subir:** validar sintaxis de los `<script>` con `node -e "new Function(...)"` y probar en navegador headless (playwright) sirviendo la carpeta con un servidor local.
- Commits en español, describiendo el cambio para el usuario.
- Fernando revisa en su celular y le habla al huésped en inglés; la app tiene textos `en` y `es` y cada texto nuevo va en los dos.

## Archivos
- `index.html` — HTML + CSS (`<style>`, arriba) + JS (`<script>`, abajo).
- `img/` — fotos WebP (`N.webp`) y logos (`logo-*.webp`, `logo-vip-white.png`). Las fotos se asignan con clases CSS `.ph-<nombre>` en la sección "Fotos" del CSS.
- `manifest.json`, `icon-*.png` — instalación como app.

## Mapa del JS (buscar por nombre)
**Datos**
- `ITEMS` — catálogo base (spa, masajes, CBD). Se le agregan más con `ITEMS.splice/push` más abajo: `spaItem(...)` (spa ampliado), `EXC` + `EXC_DESC` + `TICKET` + `LOGOS` (excursiones), `CABANA`, `PKGS` (paquetes del spa), cena Lupita.
- Campos de un ítem: `id`, `cat`, `grp`, `photo`/`sheetPhoto`/`photo2`/`gallery`, `unit` (2 = pareja), `max`, `reg` o `kreg` (precio regular por técnica), `opts` (escalones de pago/Leisure Cash, hechos con `mkOpts(promo, paso)`), `name`/`sub`/`desc` en `{en, es}`, `kinds` (tiene técnica), `noKind` (piedras calientes: sin técnica), `vipOnly`, `prefOnly`.
- `MASSAGE_KINDS` — relajante / tejido profundo.
- `DURS`, `DUR_MIN`, `VAR_LBL`, `VAR_HEAD` — grupos de duración/variantes (chips "Duration").
- `UPGRADES` — escalera de mejoras (25→50→80, pareja 50→80, cenas…).
- `COUPON`, `COUPON_ITEMS`, `couponUnit/couponPrice` — cupones del resort.
- `T` — todos los textos de interfaz `{en, es}`. `PLANS/GPLANS/SPLANS` reemplazan textos cuando VIP está apagado ("Mis planes" en vez de "Mi regalo").

**Estado**
- `S` — estado global (`S.mode` = `'vip'` | `'reg'`, `S.cart = {sel, order, cert}`, `S.will` presupuesto, `S.step` Leisure Cash, `S.sheet` ficha abierta).
- `S.cart.sel[id] = { qty, opt, kind, pk: [técnica por persona], cbd: [bool por persona] }`.
- `kindMemo`, `pkMemo`, `cbdMemo` — lo que se elige en la ficha antes de agregar.
- `withItem(cart, id, qty, extra)` — devuelve un carrito nuevo (no muta). `selOf(id)`.

**Precios**
- `regPlain`, `kindReg`, `lineReg` — precio regular. `calcVip` / `calcReg` / `costIn(mode, cart)` — lo que se paga. `promoUnit`, `unitCost`.
- `personKinds`, `personCbd`, `syncCbd`, `toggleCbd` — técnica y CBD por persona; el CBD es un ítem oculto (`cbd`) cuya cantidad se sincroniza.
- `durSwap` (cambio de duración desde la ficha) y `upgradeCart` (upsell del carrito) mueven la unidad completa: en pareja son 2 personas.

**Vistas**
- `render()` → `renderCatalog`, `renderCart`, `renderVendor`, `renderTabs`, `renderSheet`.
- Ficha de producto: `itemSheet(id)` (duración `durHTML`, técnica/CBD, descripción, galería, "También te puede gustar").
- Carrito: `cartItemsHTML`, `moneyVrow`/`regVrow` (renglón VIP o promoción, siempre en el mismo lugar), `suggestHTML` (upsells), `sumsHTML`.
- Otras fichas: `certSheet`, `compareSheet`, `shareSheet` ("Llévate tu oferta", genera imagen en canvas), guía "¿Qué se te antoja?" (`gGo`, `gAdd`).
- Clics: un solo listener con `data-*` (`data-addn`, `data-pk`, `data-kind`, `data-cbd`, `data-dur`, `data-ups`…). Busca `if (d.<nombre>)`.

## Reglas de negocio (no romper)
- Los precios de VIP Services son **tarifas de la promoción de UVC**, no "member rates". No llamarlos precios de socio.
- **Resort Coupons ≠ Leisure Cash.** Cupones: $200 por habitación, aplican **sin** VIP. Leisure Cash: aplica **con** VIP, funciona como efectivo en todo el catálogo.
- Con Leisure Cash, lo que cubre el saldo se ve "FREE" con el precio original tachado; lo que lo rebasa muestra solo la diferencia.
- Con VIP nada debe salir más caro que sin VIP con cupones.
- **Masajes:** con VIP la técnica no cambia el precio (pareja $200 / $250). Sin VIP el tejido profundo es +$10 por persona (precio regular). CBD +$35 por persona.
- El certificado de 3 días/2 noches no consume presupuesto.
- Modo vendedor libre: Leisure Cash extra con doble toque, sin pre-setup. El presupuesto del huésped es opcional y nunca bloquea.
- El toggle VIP no debe hacer que aparezcan/desaparezcan bloques: solo cambian de contenido.
- Descripciones del spa: basarse en el menú oficial de Dreams Spa by Pevonia; no prometer cosas que no estén confirmadas (p. ej. fruta y bebida en masaje en pareja).
