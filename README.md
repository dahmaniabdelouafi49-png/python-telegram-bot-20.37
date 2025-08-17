import logging
from telegram import InlineKeyboardButton, InlineKeyboardMarkup, Update
from telegram.ext import (
    Application,
    CommandHandler,
    CallbackQueryHandler,
    ContextTypes,
)
import datetime

# 🔑 التوكن الخاص بالبوت
TOKEN = "7653495962:AAF_mpEDM_2i1YTU94OjGTvwRfat_K9MJeY"

# 📢 رابط قناة البوت
BOT_CHANNEL = "https://t.me/xxcb12xx/9"

# 🗂️ قاعدة بيانات مؤقتة (في الذاكرة فقط)
users_data = {}

# ⏰ وقت الهدايا
daily_gift = {"points": 200, "hours": 24}
weekly_gift = {"points": 350, "days": 7}


# 📌 تهيئة اللوجز
logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO
)


# 🏠 /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if user_id not in users_data:
        users_data[user_id] = {
            "points": 0,
            "last_daily": None,
            "last_weekly": None,
            "orders": [],
        }

    keyboard = [
        [InlineKeyboardButton("🎁 الهدية اليومية", callback_data="daily")],
        [InlineKeyboardButton("📦 الهدية الأسبوعية", callback_data="weekly")],
        [InlineKeyboardButton("🎡 عجلة الحظ", callback_data="wheel")],
        [InlineKeyboardButton("🛠️ الخدمات المجانية", callback_data="services")],
        [InlineKeyboardButton("📋 طلباتي", callback_data="orders")],
        [InlineKeyboardButton("🎟️ استخدام كود", callback_data="use_code")],
        [InlineKeyboardButton("💰 تجميع نقاط", callback_data="collect_points")],
        [InlineKeyboardButton("📢 تمويل قناتي", callback_data="fund_channel")],
        [InlineKeyboardButton("📺 قناة البوت", url=BOT_CHANNEL)],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text("👋 مرحبا بك في البوت!\nاختر من القائمة:", reply_markup=reply_markup)


# 🎁 الهدية اليومية
async def daily_gift_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user_id = query.from_user.id
    user = users_data[user_id]

    now = datetime.datetime.now()
    if user["last_daily"] and (now - user["last_daily"]).total_seconds() < daily_gift["hours"] * 3600:
        remaining = daily_gift["hours"] * 3600 - (now - user["last_daily"]).total_seconds()
        hours, minutes = divmod(int(remaining // 60), 60)
        await query.edit_message_text(
            f"⏳ انتظر {hours} ساعة و {minutes} دقيقة للحصول على هديتك اليومية."
        )
        return

    user["points"] += daily_gift["points"]
    user["last_daily"] = now
    await query.edit_message_text(f"✅ حصلت على {daily_gift['points']} نقطة كهديتك اليومية! 🎁\nرصيدك الحالي: {user['points']} نقطة.")


# 📦 الهدية الأسبوعية
async def weekly_gift_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user_id = query.from_user.id
    user = users_data[user_id]

    now = datetime.datetime.now()
    if user["last_weekly"] and (now - user["last_weekly"]).days < weekly_gift["days"]:
        remaining_days = weekly_gift["days"] - (now - user["last_weekly"]).days
        await query.edit_message_text(
            f"⏳ انتظر {remaining_days} يوم للحصول على هديتك الأسبوعية."
        )
        return

    user["points"] += weekly_gift["points"]
    user["last_weekly"] = now
    await query.edit_message_text(f"✅ حصلت على {weekly_gift['points']} نقطة كهديتك الأسبوعية! 🎁\nرصيدك الحالي: {user['points']} نقطة.")


# 🛠️ الخدمات المجانية
async def services_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    services = [
        "مشتركين قنوات جروبات",
        "احالات بوتات",
        "مشاهدات بوستات",
        "تفاعلات بوستات",
        "متابعين انستغرام",
        "مشاهدات انستغرام",
        "لايكات انستغرام",
    ]
    keyboard = [[InlineKeyboardButton(s, callback_data=f"service_{s}")] for s in services]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await query.edit_message_text("اختر الخدمة المطلوبة:", reply_markup=reply_markup)


# 📋 طلباتي
async def orders_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user_id = query.from_user.id
    user = users_data[user_id]

    if not user["orders"]:
        await query.edit_message_text("📋 ليس لديك أي طلبات حالياً.")
    else:
        orders_text = "\n".join([f"- {o}" for o in user["orders"]])
        await query.edit_message_text(f"📋 طلباتك:\n{orders_text}")


# 🎟️ استخدام كود
async def use_code_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    await query.edit_message_text("🔑 أرسل الكود الآن للحصول على النقاط.")


# 💰 تجميع نقاط
async def collect_points_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    keyboard = [
        [InlineKeyboardButton("📢 الاشتراك بالقنوات (20 نقطة)", callback_data="join_channels")],
        [InlineKeyboardButton("🔗 رابط الدعوة (900 نقطة)", callback_data="invite_link")],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await query.edit_message_text("💰 طرق تجميع النقاط:", reply_markup=reply_markup)


# 📢 تمويل قناتي
async def fund_channel_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    await query.edit_message_text("📢 لتمويل قناتك:\n1. عين البوت مشرف في القناة/المجموعة\n2. أرسل العدد المطلوب (الحد الأدنى 20)\n3. أرسل رابط المجموعة أو معرف القناة.")


# 🎡 عجلة الحظ (بسيطة)
async def wheel_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    await query.edit_message_text("🎡 ميزة عجلة الحظ قيد التطوير.")


# 🏃‍♂️ التشغيل
def main():
    application = Application.builder().token(TOKEN).build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(CallbackQueryHandler(daily_gift_func, pattern="^daily$"))
    application.add_handler(CallbackQueryHandler(weekly_gift_func, pattern="^weekly$"))
    application.add_handler(CallbackQueryHandler(wheel_func, pattern="^wheel$"))
    application.add_handler(CallbackQueryHandler(services_func, pattern="^services$"))
    application.add_handler(CallbackQueryHandler(orders_func, pattern="^orders$"))
    application.add_handler(CallbackQueryHandler(use_code_func, pattern="^use_code$"))
    application.add_handler(CallbackQueryHandler(collect_points_func, pattern="^collect_points$"))
    application.add_handler(CallbackQueryHandler(fund_channel_func, pattern="^fund_channel$"))

    application.run_polling()


if __name__ == "__main__":
    main()
    
