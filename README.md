from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, CallbackContext
import logging
import random

# Logging sozlamalari
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

# Savollar va javoblar
savollar = [
    "📚 Dasturlash bootcamp foundation uchun qabul imtihonlar bormi?",
    "🔍 Dasturlash bootcamp foundation kursini tamomlasa 100% ish bilan ta'minlanadimi?",
    "📍 Najot Ta'lim qayerda joylashgan?",
    "📋 Najot Ta'limda qanday kurslar bor?",
    "🗣️ Til kurslari bormi?",
    "💼 Najot Ta'lim ish bilan ta'minlaydimi?",
    "🎓 Yosh chegarasi qanday?"
]

javoblar = {
    '1': "📚 **Yangilangan dasturlash bootcamp foundation** kursi uchun qabul imtihonlari 2024-yildan olib tashlangan. Biroq birinchi oyni tugatgandan keyin saralash imtihonlari bo'lib o'tadi.",
    '2': "🔍 **Najot Ta'lim** intensiv (bootcamp) kurslarni bitirgan o'quvchilarga ish taklif qilish kafolatini beradi.",
    '3': "📍 **Najot Ta'lim** ning Toshkent va Farg'ona shahrida filiallari mavjud.",
    '4': "📋 **Najot Ta'lim** da Dasturlash, Dizayn va Marketing yo'nalishida intensiv (chuqurlashtirilgan) va odatiy kurslar mavjud.",
    '5': "🗣️ Bizda til kurslari mavjud emas. «Najot Ta'lim» markazida dasturlash, grafik dizayn va marketing yo'nalishlari bo'yicha ta'lim berib kelinadi.",
    '6': "💼 **Najot Ta'lim** intensiv (bootcamp) kurslarni bitirgan o'quvchilarga ish taklif qilish kafolatini beradi.",
    '7': "🎓 Markazimizda 16 yoshdan 35 yoshgacha yosh chegarasi mavjud. 12 yoshdan 16 yoshgacha bo'lgan yoki 35 yoshdan yuqori bo'lgan o'quvchilar suhbat asosida o'qishlari mumkin bo'ladi."
}

# Dasturlash bo'yicha savollar
oyin_savollari = [
    {"savol": "Python dasturlash tilida qaysi operator tenglikni tekshiradi?", "variantlar": ["==", "=", "!="],
     "javob": "=="},
    {"savol": "C++ dasturlash tilida 'for' tsiklining standart yozuvi qanday?",
     "variantlar": ["for(i=0; i<n; i++)", "for i in range(n)", "while(i<n)"], "javob": "for(i=0; i<n; i++)"},
    {"savol": "JavaScript dasturlash tilida 'let' kalit so'zining maqsadi nima?",
     "variantlar": ["O'zgaruvchini e'lon qilish", "Funksiyani chaqirish", "Ma'lumotlarni saqlash"],
     "javob": "O'zgaruvchini e'lon qilish"},
    {"savol": "Python dasturlash tilida qaysi kalit so'z funksiyani e'lon qilish uchun ishlatiladi?",
     "variantlar": ["def", "function", "fun"], "javob": "def"},
    {"savol": "HTMLda sahifaga rasm qo'shish uchun qaysi tegi ishlatiladi?",
     "variantlar": ["<img>", "<image>", "<picture>"], "javob": "<img>"},
]


# Savollar va o'yinlar
async def start(update: Update, context: CallbackContext):
    user_first_name = update.message.from_user.first_name
    welcome_text = f"🌟 **Salom, {user_first_name}!** 🌟\n\nQuyidagi savollar ro'yxati:\n\n"

    # Savollar ro'yxatini qo'shish
    for i, savol in enumerate(savollar, 1):
        welcome_text += f"📝 {i}. {savol}\n"

    await update.message.reply_text(welcome_text, reply_markup=show_answer_buttons())


def show_answer_buttons():
    # Javobni bilish tugmalarini yaratish
    keyboard = [
        [InlineKeyboardButton(f"1-savolning javobi", callback_data='ans_1')],
        [InlineKeyboardButton(f"2-savolning javobi", callback_data='ans_2')],
        [InlineKeyboardButton(f"3-savolning javobi", callback_data='ans_3')],
        [InlineKeyboardButton(f"4-savolning javobi", callback_data='ans_4')],
        [InlineKeyboardButton(f"5-savolning javobi", callback_data='ans_5')],
        [InlineKeyboardButton(f"6-savolning javobi", callback_data='ans_6')],
        [InlineKeyboardButton(f"7-savolning javobi", callback_data='ans_7')],
        [InlineKeyboardButton(f"🎮 Dasturlash bo'yicha o'yin boshlash", callback_data='start_game')],
    ]
    return InlineKeyboardMarkup(keyboard)


async def button(update: Update, context: CallbackContext):
    query = update.callback_query
    callback_data = query.data

    if callback_data.startswith('ans_'):
        savol_id = callback_data.split('_')[1]
        javob = javoblar.get(savol_id, "❌ Kechirasiz, noto'g'ri savol raqami.")
        await query.edit_message_text(text=javob, reply_markup=InlineKeyboardMarkup(
            [[InlineKeyboardButton("🔙 Savollar ro'yxati", callback_data='show_questions')]]))
        await query.answer()
    elif callback_data == 'start_game':
        await query.answer()
        await show_game_question(update, context)


async def show_game_question(update: Update, context: CallbackContext):
    query = update.callback_query
    game_question = random.choice(oyin_savollari)
    context.user_data['current_question'] = game_question

    variantlar_buttons = [
        [InlineKeyboardButton(variant, callback_data=f"game_{variant}")] for variant in game_question["variantlar"]
    ]
    variantlar_buttons.append([InlineKeyboardButton("🔙 Savollar ro'yxati", callback_data='show_questions')])

    await query.edit_message_text(text=f"🎮 **Dasturlash bo'yicha savol** 🎮\n\n{game_question['savol']}",
                                  reply_markup=InlineKeyboardMarkup(variantlar_buttons))


async def check_game_answer(update: Update, context: CallbackContext):
    query = update.callback_query
    callback_data = query.data.split('_')[1]
    game_question = context.user_data.get('current_question')

    if game_question and callback_data == game_question["javob"]:
        await query.edit_message_text(
            text=f"✅ To'g'ri javob!\n\n{game_question['savol']}\nJavob: {game_question['javob']}")
    else:
        await query.edit_message_text(
            text=f"❌ Noto'g'ri javob!\n\n{game_question['savol']}\nTo'g'ri javob: {game_question['javob']}")

    await query.message.reply_text("🔙 Savollar ro'yxati", reply_markup=InlineKeyboardMarkup(
        [[InlineKeyboardButton("Savollar ro'yxati", callback_data='show_questions')]]))
    await query.answer()


async def show_questions(update: Update, context: CallbackContext):
    query = update.callback_query
    user_first_name = query.from_user.first_name
    welcome_text = f"🌟 **Salom, {user_first_name}!** 🌟\n\nQuyidagi savollar ro'yxati:\n\n"

    for i, savol in enumerate(savollar, 1):
        welcome_text += f"📝 {i}. {savol}\n"

    await query.edit_message_text(welcome_text, reply_markup=show_answer_buttons())
    await query.answer()


def main():
    application = Application.builder().token("7394979199:AAGV7sEhHk1Ya7MmBYYaqgpPwWpxKYCq45Y").build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(CallbackQueryHandler(button, pattern='^ans_'))
    application.add_handler(CallbackQueryHandler(show_questions, pattern='^show_questions$'))
    application.add_handler(CallbackQueryHandler(button, pattern='^start_game$'))
    application.add_handler(CallbackQueryHandler(check_game_answer, pattern='^game_'))

    application.run_polling()


if __name__ == '__main__':
    main()
