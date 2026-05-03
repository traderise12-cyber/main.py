# main.py import asyncio
from aiogram import Bot, Dispatcher, types, F
from aiogram.filters import Command
from aiogram.utils.keyboard import InlineKeyboardBuilder

# --- НАСТРОЙКИ ---
TOKEN = "6523793120"
ADMIN_ID = 6523793120  # Твой ID, чтобы бот присылал тебе заказы

bot = Bot(token=TOKEN)
dp = Dispatcher()

# --- СПИСОК УСЛУГ ---
SERVICES = {
    "duel_8": {"name": "Дуэль (15⭐️)", "price": "8г"},
    "friends_15": {"name": "Друзья (15⭐️)", "price": "15г"},
    "duel_video": {"name": "Дуэль на ролик (15⭐️)", "price": "20г"},
    "mod_stream": {"name": "Модерка на стриме (25⭐️)", "price": "50г"}
}

# --- КЛАВИАТУРЫ ---
def main_menu():
    builder = InlineKeyboardBuilder()
    builder.row(types.InlineKeyboardButton(text="⚔️ Дуэли", callback_data="cat_duels"))
    builder.row(types.InlineKeyboardButton(text="🎭 Услуги и Друзья", callback_data="cat_other"))
    builder.row(types.InlineKeyboardButton(text="👨‍💻 Владелец", url="https://t.me"))
    return builder.as_markup()

# --- ХЕНДЛЕРЫ ---
@dp.message(Command("start"))
async def start(message: types.Message):
    await message.answer(
        f"Привет, {message.from_user.first_name}! 👋\nВыбирай нужную услугу в меню ниже:",
        reply_markup=main_menu()
    )

@dp.callback_query(F.data == "cat_duels")
async def show_duels(callback: types.CallbackQuery):
    builder = InlineKeyboardBuilder()
    builder.row(types.InlineKeyboardButton(text="Дуэль 15⭐️ — 8г", callback_data="buy_duel_8"))
    builder.row(types.InlineKeyboardButton(text="Дуэль на ролик 15⭐️ — 20г", callback_data="buy_duel_video"))
    builder.row(types.InlineKeyboardButton(text="⬅️ Назад", callback_data="back"))
    
    await callback.message.edit_text("Выбери тип дуэли:", reply_markup=builder.as_markup())

@dp.callback_query(F.data == "cat_other")
async def show_other(callback: types.CallbackQuery):
    builder = InlineKeyboardBuilder()
    builder.row(types.InlineKeyboardButton(text="Друзья 15⭐️ — 15г", callback_data="buy_friends_15"))
    builder.row(types.InlineKeyboardButton(text="Модерка 25⭐️ — 50г", callback_data="buy_mod_stream"))
    builder.row(types.InlineKeyboardButton(text="⬅️ Назад", callback_data="back"))
    
    await callback.message.edit_text("Дополнительные услуги:", reply_markup=builder.as_markup())

@dp.callback_query(F.data.startswith("buy_"))
async def handle_purchase(callback: types.CallbackQuery):
    service_key = callback.data.replace("buy_", "")
    service = SERVICES[service_key]
    user = callback.from_user
    
    # Отправляем уведомление тебе (админу)
    await bot.send_message(
        ADMIN_ID,
        f"🛍 <b>Новый заказ!</b>\n\n"
        f"Услуга: {service['name']}\n"
        f"Цена: {service['price']}\n"
        f"Покупатель: @{user.username if user.username else 'нет юзернейма'}\n"
        f"ID: <code>{user.id}</code>",
        parse_mode="HTML"
    )
    
    await callback.answer("✅ Заявка отправлена! Жди сообщения от админа.", show_alert=True)

@dp.callback_query(F.data == "back")
async def back_to_menu(callback: types.CallbackQuery):
    await callback.message.edit_text("Выбирай нужную услугу в меню ниже:", reply_markup=main_menu())

async def main():
    print("🔥 Бот-магазин запущен!")
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
