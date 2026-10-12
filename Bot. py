import asyncio
import os
import sqlite3

from aiogram import Bot, Dispatcher, F
from aiogram.filters import CommandStart
from aiogram.types import (CallbackQuery, InlineKeyboardButton,
                           InlineKeyboardMarkup, Message)

TOKEN = os.getenv("BOT_TOKEN")
ADMIN_ID = int(os.getenv("ADMIN_ID", "0"))
CHANNELS = [c.strip() for c in os.getenv("CHANNELS", "").split(",") if c.strip()]

db = sqlite3.connect("movies.db")
db.execute("CREATE TABLE IF NOT EXISTS movies (code TEXT PRIMARY KEY, file_id TEXT, kind TEXT)")
db.commit()

bot = Bot(TOKEN)
dp = Dispatcher()


async def is_subscribed(user_id: int) -> bool:
    for ch in CHANNELS:
        try:
            m = await bot.get_chat_member(ch, user_id)
            if m.status in ("left", "kicked"):
                return False
        except Exception:
            return False
    return True


def channels_kb() -> InlineKeyboardMarkup:
    rows = [[InlineKeyboardButton(text=f"📢 {c}", url=f"https://t.me/{c.lstrip('@')}")]
            for c in CHANNELS]
    rows.append([InlineKeyboardButton(text="✅ Tekshirish", callback_data="check")])
    return InlineKeyboardMarkup(inline_keyboard=rows)


@dp.message(CommandStart())
async def start(msg: Message):
    if await is_subscribed(msg.from_user.id):
        await msg.answer("✅ Obuna tasdiqlandi. Kino kodini yuboring:")
    else:
        await msg.answer("Kinoni olish uchun quyidagi kanallarga obuna bo'ling:",
                         reply_markup=channels_kb())


@dp.callback_query(F.data == "check")
async def check(cb: CallbackQuery):
    if await is_subscribed(cb.from_user.id):
        await cb.message.answer("✅ Obuna tasdiqlandi. Kino kodini yuboring:")
    else:
        await cb.answer("❌ Hali hamma kanalga obuna bo'lmadingiz!", show_alert=True)


@dp.message(F.from_user.id == ADMIN_ID, F.video | F.document)
async def add_movie(msg: Message):
    code = (msg.caption or "").strip()
    if not code:
        return await msg.answer("❗ Videoning izohiga kod yozing (masalan: 123)")
    file_id = msg.video.file_id if msg.video else msg.document.file_id
    kind = "video" if msg.video else "document"
    db.execute("INSERT OR REPLACE INTO movies VALUES (?,?,?)", (code, file_id, kind))
    db.commit()
    await msg.answer(f"✅ Saqlandi. Kod: {code}")


@dp.message(F.text)
async def get_movie(msg: Message):
    if not await is_subscribed(msg.from_user.id):
        return await msg.answer("Avval kanallarga obuna bo'ling:", reply_markup=channels_kb())
    row = db.execute("SELECT file_id, kind FROM movies WHERE code=?",
                     (msg.text.strip(),)).fetchone()
    if not row:
        return await msg.answer("❌ Bunday kodli kino topilmadi.")
    file_id, kind = row
    if kind == "video":
        await msg.answer_video(file_id, caption="🎬 Yoqimli tomosha!")
    else:
        await msg.answer_document(file_id, caption="🎬 Yoqimli tomosha!")


async def main():
    await dp.start_polling(bot)


if __name__ == "__main__":
    asyncio.run(main())
