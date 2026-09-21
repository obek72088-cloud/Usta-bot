import asyncio, os, sqlite3
from aiohttp import web
from aiogram import Bot, Dispatcher, F
from aiogram.filters import CommandStart
from aiogram.types import Message, CallbackQuery, InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.fsm.state import State, StatesGroup
from aiogram.fsm.context import FSMContext
from aiogram.fsm.storage.memory import MemoryStorage

TOKEN = os.getenv("BOT_TOKEN")
DB = "usta_bot.db"

def db():
    con = sqlite3.connect(DB)
    con.row_factory = sqlite3.Row
    return con

def init_db():
    con = db()
    con.execute("""CREATE TABLE IF NOT EXISTS users(
        tg_id INTEGER PRIMARY KEY, name TEXT, phone TEXT, role TEXT DEFAULT 'customer'
    )""")
    con.execute("""CREATE TABLE IF NOT EXISTS masters(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        tg_id INTEGER UNIQUE, name TEXT, phone TEXT, category TEXT,
        city TEXT, district TEXT, experience INTEGER, description TEXT,
        photo TEXT, rating REAL DEFAULT 0, reviews INTEGER DEFAULT 0,
        verified INTEGER DEFAULT 0, created_at TEXT DEFAULT CURRENT_TIMESTAMP
    )""")
    con.commit(); con.close()

class RegisterMaster(StatesGroup):
    name = State(); phone = State(); category = State(); city = State()
    district = State(); experience = State(); description = State(); photo = State()

class SearchMaster(StatesGroup):
    category = State(); city = State(); district = State()

def main_kb():
    return InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text="🔎 Usta topish", callback_data="search")],
        [InlineKeyboardButton(text="👨‍🔧 Usta bo‘lib qo‘shilish", callback_data="join")],
        [InlineKeyboardButton(text="👤 Profilim", callback_data="profile")],
        [InlineKeyboardButton(text="ℹ️ Loyiha haqida", callback_data="about")]
    ])

dp = Dispatcher(storage=MemoryStorage())

@dp.message(CommandStart())
async def start(m: Message):
    con=db(); con.execute("INSERT OR IGNORE INTO users(tg_id,name) VALUES(?,?)",(m.from_user.id,m.from_user.full_name)); con.commit(); con.close()
    await m.answer("👋 <b>USTA BOT</b>\n\nO‘z hududingizdagi ustalarni toping yoki o‘zingizni bepul usta sifatida qo‘shing.", reply_markup=main_kb(), parse_mode="HTML")

@dp.callback_query(F.data=="about")
async def about(c: CallbackQuery):
    await c.message.edit_text("🚀 <b>USTA BOT</b>\n\nHozircha barcha ustalar uchun bepul.\nKeyinchalik qo‘shimcha PRO funksiyalar ishga tushiriladi.", reply_markup=InlineKeyboardMarkup(inline_keyboard=[[InlineKeyboardButton(text="⬅️ Orqaga", callback_data="home")]]), parse_mode="HTML")
    await c.answer()

@dp.callback_query(F.data=="home")
async def home(c: CallbackQuery):
    await c.message.edit_text("🏠 Bosh menyu", reply_markup=main_kb()); await c.answer()

@dp.callback_query(F.data=="join")
async def join(c: CallbackQuery, state: FSMContext):
    await state.set_state(RegisterMaster.name); await c.message.edit_text("1/8 👤 Ismingizni yozing:"); await c.answer()

@dp.message(RegisterMaster.name)
async def r_name(m,state):
    await state.update_data(name=m.text); await state.set_state(RegisterMaster.phone); await m.answer("2/8 📞 Telefon raqamingizni yuboring (masalan +998901234567):")

@dp.message(RegisterMaster.phone)
async def r_phone(m,state):
    await state.update_data(phone=m.text); await state.set_state(RegisterMaster.category); await m.answer("3/8 🔧 Xizmatingizni yozing:\nMasalan: Santexnik, Elektrik, Konditsioner")

@dp.message(RegisterMaster.category)
async def r_cat(m,state):
    await state.update_data(category=m.text); await state.set_state(RegisterMaster.city); await m.answer("4/8 📍 Shahar/tumanni yozing:")

@dp.message(RegisterMaster.city)
async def r_city(m,state):
    await state.update_data(city=m.text); await state.set_state(RegisterMaster.district); await m.answer("5/8 📍 Mahalla yoki hududni yozing:")

@dp.message(RegisterMaster.district)
async def r_dist(m,state):
    await state.update_data(district=m.text); await state.set_state(RegisterMaster.experience); await m.answer("6/8 👨‍🔧 Tajribangiz necha yil?")

@dp.message(RegisterMaster.experience)
async def r_exp(m,state):
    try: int(m.text)
    except: return await m.answer("Faqat yil sonini yozing. Masalan: 5")
    await state.update_data(experience=int(m.text)); await state.set_state(RegisterMaster.description); await m.answer("7/8 📝 O‘zingiz va xizmatlaringiz haqida qisqacha yozing:")

@dp.message(RegisterMaster.description)
async def r_desc(m,state):
    await state.update_data(description=m.text); await state.set_state(RegisterMaster.photo); await m.answer("8/8 📷 Ishingizdan 1 ta rasm yuboring yoki /skip yozing.")

@dp.message(RegisterMaster.photo, F.photo)
async def r_photo(m,state):
    await state.update_data(photo=m.photo[-1].file_id); await save_master(m,state)

@dp.message(RegisterMaster.photo, F.text=="/skip")
async def r_skip(m,state):
    await state.update_data(photo=None); await save_master(m,state)

async def save_master(m,state):
    d=await state.get_data(); con=db()
    con.execute("""INSERT INTO masters(tg_id,name,phone,category,city,district,experience,description,photo)
        VALUES(?,?,?,?,?,?,?,?,?) ON CONFLICT(tg_id) DO UPDATE SET name=excluded.name,phone=excluded.phone,
        category=excluded.category,city=excluded.city,district=excluded.district,
        experience=excluded.experience,description=excluded.description,photo=excluded.photo""",
        (m.from_user.id,d["name"],d["phone"],d["category"],d["city"],d["district"],d["experience"],d["description"],d.get("photo")))
    con.commit(); con.close(); await state.clear(); await m.answer("✅ Profilingiz muvaffaqiyatli qo‘shildi!\n\nHozircha xizmat bepul.", reply_markup=main_kb())

@dp.callback_query(F.data=="search")
async def search(c,state):
    await state.set_state(SearchMaster.category); await c.message.edit_text("🔎 Qaysi usta kerak?\n\nMasalan: santexnik, elektrik, konditsioner"); await c.answer()

@dp.message(SearchMaster.category)
async def s_cat(m,state):
    await state.update_data(category=m.text); await state.set_state(SearchMaster.city); await m.answer("📍 Qaysi shahar/tuman?")

@dp.message(SearchMaster.city)
async def s_city(m,state):
    await state.update_data(city=m.text); await state.set_state(SearchMaster.district); await m.answer("📍 Mahalla/hudud (yoki 'farqi yo‘q'):")

@dp.message(SearchMaster.district)
async def s_dist(m,state):
    d=await state.get_data(); district=m.text; con=db()
    rows=con.execute("""SELECT * FROM masters WHERE lower(category) LIKE lower(?)
        AND lower(city) LIKE lower(?) AND (lower(district) LIKE lower(?) OR lower(?)='farqi yo‘q')
        ORDER BY verified DESC, rating DESC, reviews DESC LIMIT 10""",
        (f"%{d['category']}%",f"%{d['city']}%",f"%{district}%",district)).fetchall()
    con.close(); await state.clear()
    if not rows: return await m.answer("😕 Hozircha mos usta topilmadi.", reply_markup=main_kb())
    for r in rows:
        text=f"👨‍🔧 <b>{r['name']}</b>\n🔧 {r['category']}\n📍 {r['city']}, {r['district']}\n⭐ {r['rating']:.1f} ({r['reviews']})\n🧰 Tajriba: {r['experience']} yil\n\n{r['description']}"
        kb=InlineKeyboardMarkup(inline_keyboard=[[InlineKeyboardButton(text="📞 Telefon", callback_data=f"phone:{r['id']}")]])
        if r["photo"]:
            try: await m.answer_photo(r["photo"],caption=text,reply_markup=kb,parse_mode="HTML")
            except: await m.answer(text,reply_markup=kb,parse_mode="HTML")
        else: await m.answer(text,reply_markup=kb,parse_mode="HTML")
    await m.answer("🏠 Bosh menyu",reply_markup=main_kb())

@dp.callback_query(F.data.startswith("phone:"))
async def phone(c):
    mid=int(c.data.split(":")[1]); con=db(); r=con.execute("SELECT phone FROM masters WHERE id=?",(mid,)).fetchone(); con.close()
    if not r: return await c.answer("Usta topilmadi.", show_alert=True)
    await c.message.answer(f"📞 Usta telefoni: <code>{r['phone']}</code>",parse_mode="HTML"); await c.answer()

@dp.callback_query(F.data=="profile")
async def profile(c):
    con=db(); r=con.execute("SELECT * FROM masters WHERE tg_id=?",(c.from_user.id,)).fetchone(); con.close()
    if not r: return await c.message.edit_text("Siz hali usta sifatida ro‘yxatdan o‘tmagansiz.",reply_markup=main_kb())
    await c.message.edit_text(f"👨‍🔧 <b>{r['name']}</b>\n🔧 {r['category']}\n📍 {r['city']}, {r['district']}\n🧰 {r['experience']} yil tajriba\n⭐ {r['rating']:.1f} ({r['reviews']} baho)",reply_markup=main_kb(),parse_mode="HTML")
    await c.answer()

async def health(request):
    return web.Response(text="OK")

async def run_web_server():
    app = web.Application()
    app.router.add_get("/", health)
    app.router.add_get("/health", health)
    runner = web.AppRunner(app)
    await runner.setup()
    port = int(os.getenv("PORT", "10000"))
    site = web.TCPSite(runner, "0.0.0.0", port)
    await site.start()
    return runner

async def main():
    if not TOKEN:
        raise RuntimeError("BOT_TOKEN environment variable is missing.")
    init_db()
    bot = Bot(TOKEN)
    runner = await run_web_server()
    try:
        await dp.start_polling(bot)
    finally:
        await bot.session.close()
        await runner.cleanup()

if __name__=="__main__":
    asyncio.run(main())
