# ID-Analyzer-AI-Bot
import datetime
import telebot
from hijri_converter import convert

# 🛡️ ضع توكن البوت هنا
BOT_TOKEN = "7601114915:AAGdt7AzHdmolKkHD3pIeu48M9utl_xzrys"
bot = telebot.TeleBot(BOT_TOKEN)

# 🏡 أكواد المحافظات المصرية وعدد سكانها (تقديري)
gov_data = {
    "01": ("القاهرة", 10230350), "02": ("الإسكندرية", 5412175), "03": ("بورسعيد", 768750),
    "04": ("السويس", 781370), "11": ("دمياط", 1522000), "12": ("الدقهلية", 6832300),
    "13": ("الشرقية", 7760500), "14": ("القليوبية", 6019500), "15": ("كفر الشيخ", 3524600),
    "16": ("الغربية", 5212500), "17": ("المنوفية", 4096400), "18": ("البحيرة", 6554500),
    "19": ("الإسماعيلية", 1388000), "21": ("الجيزة", 9243200), "22": ("بني سويف", 3112700),
    "23": ("الفيوم", 3641300), "24": ("المنيا", 5913000), "25": ("أسيوط", 4768700),
    "26": ("سوهاج", 5256300), "27": ("قنا", 3565200), "28": ("أسوان", 1532500),
    "29": ("الأقصر", 1281900), "31": ("البحر الأحمر", 395000), "32": ("الوادى الجديد", 260000),
    "33": ("مطروح", 520000), "34": ("شمال سيناء", 450000), "35": ("جنوب سيناء", 107000)
}

# 🔮 الأبراج الفلكية
zodiac_signs = {
    (3, 21): "الحمل", (4, 20): "الثور", (5, 21): "الجوزاء", (6, 21): "السرطان",
    (7, 23): "الأسد", (8, 23): "العذراء", (9, 23): "الميزان", (10, 23): "العقرب",
    (11, 22): "القوس", (12, 22): "الجدي", (1, 20): "الدلو", (2, 19): "الحوت"
}

# 🐉 الأبراج الصينية
chinese_zodiac_animals = [
    "الفأر", "الثور", "النمر", "الأرنب", "التنين", "الأفعى",
    "الحصان", "الماعز", "القرد", "الديك", "الكلب", "الخنزير"
]

# 🔥 الأجيال (Generation)
def determine_generation(year):
    if year >= 2013:
        return "جيل ألفا (Alpha)"
    elif year >= 1997:
        return "جيل زد (Gen Z)"
    elif year >= 1981:
        return "جيل الألفية (Millennials / Gen Y)"
    elif year >= 1965:
        return "جيل إكس (Gen X)"
    else:
        return "جيل الطفرة السكانية (Boomers)"

# 🏷️ الفئات العمرية
def determine_age_group(age):
    if age < 13:
        return "طفل 👶"
    elif age < 18:
        return "مراهق 🧑‍🎓"
    elif age < 40:
        return "شاب 🏃‍♂️"
    elif age < 60:
        return "كهل 👨‍🦳"
    else:
        return "مسن 👴"

# 🎯 استقبال رقم البطاقة من المستخدم
@bot.message_handler(commands=['start'])
def send_welcome(message):
    bot.reply_to(message, "👋 أهلاً بك! أرسل رقم بطاقتك القومية (14 رقمًا) للحصول على معلوماتك.")

@bot.message_handler(func=lambda message: True)
def process_national_id(message):
    national_id = message.text.strip()

    if len(national_id) != 14 or not national_id.isdigit():
        bot.reply_to(message, "❌ خطأ: تأكد من إدخال رقم بطاقة صحيح مكون من 14 رقمًا.")
        return

    # استخراج البيانات من رقم البطاقة
    century_code = national_id[0]  
    year_short = int(national_id[1:3])
    month_gregorian = int(national_id[3:5])
    day_gregorian = int(national_id[5:7])

    if century_code == "2":
        year_gregorian = 1900 + year_short
    elif century_code == "3":
        year_gregorian = 2000 + year_short
    else:
        bot.reply_to(message, "❌ خطأ: رقم البطاقة غير صحيح أو لا يتبع النظام المصري.")
        return

    birth_date = datetime.date(year_gregorian, month_gregorian, day_gregorian)
    hijri_date = convert.Gregorian(year_gregorian, month_gregorian, day_gregorian).to_hijri()
    now = datetime.datetime.now()
    age_years = now.year - birth_date.year
    age_months = (now.month - birth_date.month) % 12
    age_days = (now.day - birth_date.day) % 30
    age_hours = (now - datetime.datetime(year_gregorian, month_gregorian, day_gregorian)).total_seconds() // 3600
    age_minutes = age_hours * 60
    age_seconds = age_minutes * 60

    zodiac = next(z for (m, d), z in reversed(zodiac_signs.items()) if (month_gregorian, day_gregorian) >= (m, d))
    chinese_zodiac = chinese_zodiac_animals[year_gregorian % 12]

    gov_code = national_id[7:9]
    birth_place, population = gov_data.get(gov_code, ("غير معروف", "غير متوفر"))

    birth_order = int(national_id[9:13])
    world_population = 6_000_000_000 + birth_order * 3  
    egypt_population = 70_000_000 + birth_order  

    generation = determine_generation(year_gregorian)
    age_group = determine_age_group(age_years)

    response = f"""
📌 *معلومات الشخص*  
📅 *يوم الميلاد:* {birth_date.strftime('%A')}  
📅 *التاريخ الميلادي:* {day_gregorian}-{month_gregorian}-{year_gregorian}  
🌙 *الشهر الهجري:* {hijri_date.month}  
🎂 *العمر:* {age_years} سنة، {age_months} شهر، {age_days} يوم  
⏳ *بالتفصيل:* {age_hours:,} ساعة، {age_minutes:,} دقيقة، {age_seconds:,} ثانية  
🔮 *البرج الفلكي:* {zodiac}  
🐲 *البرج الصيني:* {chinese_zodiac}  
📍 *محافظة الميلاد:* {birth_place} (عدد السكان: {population:,})  
🎯 *ترتيب الولادة:*  
   🏆 *عالميًا:* {world_population:,}  
   🇪🇬 *في مصر:* {egypt_population:,}  
👶 *الفئة العمرية:* {age_group}  
🕰️ *الجيل:* {generation}  
"""







    bot.reply_to(message, response)

# 🚀 تشغيل البوت
print("✅ البوت يعمل الآن...")
bot.polling()
