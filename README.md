import telebot
from telebot.types import ReplyKeyboardMarkup, KeyboardButton
import re

# ضع التوكن الخاص بك هنا
TOKEN = '7683296600:AAECcnfSHc8TfpbJm45-I0objJdsKq3z8A4'  # استبدل هذا بالتوكن الفعلي
ADMIN = '6725533553'  # استبدل هذا بمعرف الأدمن

bot = telebot.TeleBot(TOKEN)

# رد على أوامر السكري
@bot.message_handler(commands=['سكري', 'Secre'])
def test(message):
    bot.send_message(chat_id=message.chat.id, text='ماذا؟')

# رفع أدمن
@bot.message_handler(commands=['رفع أدمن.'])
def pro(message):
    admin = str(message.from_user.id)
    if ADMIN == admin:
        if message.reply_to_message:
            user = message.reply_to_message.from_user.id
            userN = message.reply_to_message.from_user.first_name
            bot.promote_chat_member(message.chat.id, user, can_manage_chat=True)
            bot.send_message(chat_id=message.chat.id, text=f'مبروك {userN}')
        else:
            bot.reply_to(message, 'يرجى الرد على الرسالة لتتمكن من رفع المستخدم')

# عزل أدمن
@bot.message_handler(commands=['عزل.'])
def dem(message):
    admin = str(message.from_user.id)
    if ADMIN == admin:
        if message.reply_to_message:
            user = message.reply_to_message.from_user.id
            userN = message.reply_to_message.from_user.first_name
            bot.promote_chat_member(message.chat.id, user, can_manage_chat=False)
            bot.send_message(chat_id=message.chat.id, text=f'ياللأسف يبدو أنك نزلت من مكانك! {userN}')
        else:
            bot.reply_to(message, 'يرجى الرد على الرسالة لتتمكن من عزل المستخدم')

# طرد مستخدم
@bot.message_handler(commands=['طرد.'])
def طرد_المستخدم(message):
    admin = str(message.from_user.id)
    if ADMIN == admin:
        if message.reply_to_message:
            user_id = message.reply_to_message.from_user.id
            bot.kick_chat_member(message.chat.id, user_id)
            bot.send_message(chat_id=message.chat.id, text=f'لقد تم طرده {user_id}')
        else:
            bot.reply_to(message, 'يرجى الرد على الرسالة لتتمكن من طرد المستخدم')

# مسح الروابط
pattern = re.compile(r'https?://[^\s]+')

@bot.message_handler(content_types=['text'])
def delete(message):
    user_id = message.from_user.id
    if user_id != ADMIN:  # تحقق إذا كان المستخدم ليس الأدمن
        if pattern.search(message.text):
            bot.delete_message(chat_id=message.chat.id, message_id=message.message_id)

# التعامل مع الأوامر الأخرى
@bot.message_handler(commands=['دانجن'])
def test(message):
    bot.send_photo(message.chat.id, photo='https://t.me/WutheringWavesOtooma/13')  # ضع الرابط ه
bot.polling()
