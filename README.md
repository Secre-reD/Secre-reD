import telebot
from telebot import types
import random
import time

# ضع التوكن الخاص بك هنا
TOKEN = "7683296600:AAECcnfSHc8TfpbJm45-I0objJdsKq3z8A4"
bot = telebot.TeleBot(TOKEN)

# قائمة المشرفين (ستكون ديناميكية حسب المجموعة)
admins = []

# قائمة العقوبات والإنذارات
warnings = {}

# الردود العشوائية
response_hello = ["وعليكم السلام", "وعليكم السلام ورحمة الله وبركاته"]
response_greetings = ["وأين السلام في هذا الكلام؟"]

# الردود الساخره لسكري
sarcastic_responses = ["هاها، هل هذا ما كنت تتوقعه؟", "لم يكن ذلك مثيرًا كما كنت تأمل", "يا إلهي! كل هذا كان ممتعًا جدًا!"]

# الردود للملل
bored_responses = ["ماذا تفعل؟", "هل تريد شيء مختلف؟", "حسنًا، إذا كنت تشعر بالملل، فأنت لست وحدك."]

# الردود على القوانين
laws_text = """اترك_مكان_القونين_فارغ
1. لا تسب أو تلعن.
2. لا تكرر الرسائل أو الملصقات.
3. لا تصحح أخطاء الآخرين بطريقة غير لائقة.
4. لا تكرر الرسائل بشكل مفرط."""

# تحديد نسبة الرد على الأعضاء العاديين
response_probability = 0.8

# الوظيفة الأساسية عند إرسال رسالة جديدة
@bot.message_handler(commands=["start"])
def send_welcome(message):
    bot.reply_to(message, "مرحبًا! أنا بوت سكري، كيف يمكنني مساعدتك اليوم؟")

# ردود على السلام
@bot.message_handler(func=lambda message: "السلام عليكم" in message.text or "وعليكم السلام" in message.text)
def reply_to_hello(message):
    bot.reply_to(message, random.choice(response_hello))

# ردود على التحية
@bot.message_handler(func=lambda message: any(word in message.text for word in ["مرحبا", "هلو", "صباح الخير", "مساء الخير"]))
def reply_to_greetings(message):
    bot.reply_to(message, random.choice(response_greetings))

# مسح الروابط والملفات
@bot.message_handler(content_types=['text', 'document', 'photo', 'audio', 'video'])
def handle_attachments(message):
    if message.from_user.id not in admins:
        bot.delete_message(message.chat.id, message.message_id)

# ملف العضو
@bot.message_handler(commands=["ملفي"])
def show_profile(message):
    user_info = f"معرفك: {message.from_user.id}\nاسمك: {message.from_user.first_name}\nعدد الرسائل المرسلة: {random.randint(1, 100)}\n"
    bot.reply_to(message, user_info)

# إرسال اقتباسات
@bot.message_handler(commands=["اقتباسات"])
def send_quotes(message):
    quotes = ["النجاح ليس نهائيًا، الفشل ليس قاتلاً: الشجاعة تهم.", "عش كل يوم وكأنه آخر يوم في حياتك."]
    bot.reply_to(message, random.choice(quotes))

# أمر "سكري"
@bot.message_handler(commands=["سكري"])
def sarcastic_reply(message):
    bot.reply_to(message, random.choice(sarcastic_responses))

# أمر "ملل"
@bot.message_handler(commands=["ملل"])
def bored_reply(message):
    bot.reply_to(message, random.choice(bored_responses))

# إضافة مشرف
@bot.message_handler(commands=["رفع مشرف"])
def add_admin(message):
    if message.from_user.id in admins:
        new_admin_id = message.reply_to_message.from_user.id
        admins.append(new_admin_id)
        bot.reply_to(message, "تم رفعه كمشرف")
    else:
        bot.reply_to(message, "أنت لست مشرفًا!")

# عقوبات وإنذارات
@bot.message_handler(commands=["أنذار"])
def issue_warning(message):
    user_id = message.reply_to_message.from_user.id
    if user_id not in warnings:
        warnings[user_id] = 1
    else:
        warnings[user_id] += 1

    if warnings[user_id] >= 4:
        bot.kick_chat_member(message.chat.id, user_id)
        bot.reply_to(message, f"تم حظر {message.reply_to_message.from_user.first_name} بعد 4 إنذارات!")
    else:
        bot.reply_to(message, f"تم إعطاء إنذار {warnings[user_id]} لـ {message.reply_to_message.from_user.first_name}")

# أمر إزالة الإنذارات
@bot.message_handler(commands=["إزالة_الإنذارات"])
def remove_warning(message):
    user_id = message.reply_to_message.from_user.id
    if user_id in warnings:
        warnings[user_id] = 0
        bot.reply_to(message, f"تم إزالة الإنذارات عن {message.reply_to_message.from_user.first_name}")
    else:
        bot.reply_to(message, "لا يوجد إنذارات لهذا الشخص!")

# أوامر المشرفين
@bot.message_handler(commands=["الأوامر"])
def show_commands(message):
    if message.from_user.id in admins:
        bot.reply_to(message, "\n".join(admin_commands.values()))
    else:
        bot.reply_to(message, "أنت لست مشرفًا!")

# الرد على القوانين
@bot.message_handler(func=lambda message: "قوانين" in message.text or "القوانين" in message.text)
def show_laws(message):
    bot.reply_to(message, laws_text)

# الأمر "همس"
@bot.message_handler(func=lambda message: "اهمس" in message.text)
def whisper(message):
    bot.reply_to(message, "قل لي ما تريد همسه!")
    bot.register_next_step_handler(message, process_whisper)

def process_whisper(message):
    bot.send_message(message.chat.id, f"الهمسة: {message.text} (من {message.from_user.first_name} إلى {message.reply_to_message.from_user.first_name})")

# حفظ التعديلات على GitHub و Replit
# يجب عليك تحميل الكود على GitHub ووصله بـ Replit لتشغيل البوت بشكل مستمر

# معدل الرد على الأعضاء العاديين
def should_reply():
    return random.random() <= response_probability

# تشغيل البوت
bot.polling(none_stop=True)
