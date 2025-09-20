# import telebot
from telebot import types
from datetime import datetime

TOKEN = "8498650096:AAEp7r9ZJklsWh2MGNC-HL6GpNs7jpVyxaQ"
ADMIN_ID = 7794500215

bot = telebot.TeleBot(TOKEN, parse_mode="Markdown")

# بيانات مؤقتة
user_orders = {}
user_temp = {}
pending_orders = {}  # لتخزين كل الطلبات الجديدة

# أسعار الألعاب
freefire_packs = {
    "110💎": 11500,
    "500💎": 57000,
    "341💎": 34500,
    "1032💎": 114000,
    "1166💎": 126000,
    "عضوية شهرية💎": 13100,
    "عضوية اسبوعية": 30000
}

pubg_packs = {
    "60♣️": 11000,
    "325♣️": 55000,
    "660♣️": 110000,
    "1800♣️": 290000
}

# دوال
def apply_discount(price):
    today = datetime.today()
    if today.month == 9 and today.day in [2,3]:
        return int(price * 0.95)
    return price

def main_menu(chat_id):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    markup.add("💎 FREE FIRE", "🎯 PUBG")
    markup.add("🔥 عروضنا", "📞 تواصل", "👥 مجموعتنا", "💰 طريقة الشحن")
    bot.send_message(chat_id, "🏰 القائمة الرئيسية", reply_markup=markup)

def shipping_menu():
    m = types.ReplyKeyboardMarkup(resize_keyboard=True)
    m.add("📲 حساب سيرياتيل كاش")
    m.add("💵 بدون سيرياتيل كاش")
    m.add("⬅ العودة")
    return m

# أزرار الطلب بعد إدخال الأيدي
def order_id_menu():
    markup = types.InlineKeyboardMarkup()
    markup.add(
        types.InlineKeyboardButton("✅ تاكيد الطلب", callback_data="confirm_order"),
    )
    markup.add(
        types.InlineKeyboardButton("تغيير الفئة", callback_data="change_pack"),
        types.InlineKeyboardButton("تغيير الايدي", callback_data="change_id")
    )
    markup.add(
        types.InlineKeyboardButton("❌ الغاء الطلب", callback_data="cancel_order")
    )
    return markup

# /start
@bot.message_handler(commands=['start'])
def start_cmd(message):
    main_menu(message.chat.id)

# التعامل مع الرسائل
@bot.message_handler(func=lambda m: True)
def handle_messages(message):
    chat_id = message.chat.id
    text = (message.text or "").strip()

    # رجوع
    if text == "⬅ العودة":
        main_menu(chat_id)
        return

    # عروضنا
    if text == "🔥 عروضنا":
        today = datetime.today()
        if today.month == 9 and today.day in [2,3]:
            bot.send_message(chat_id, "🔥 خصم 5٪ على جميع الجواهر والشدات اليوم!")
        else:
            bot.send_message(chat_id, "❌ لا توجد عروض حالية")
        return

    # تواصل
    if text == "📞 تواصل":
        bot.send_message(chat_id, "📞 للتواصل مع الأدمن: @IVAIXIAIVAIROI")
        return

    # مجموعتنا
    if text == "👥 مجموعتنا":
        bot.send_message(chat_id, "👥 انضم لمجموعتنا:\nhttps://chat.whatsapp.com/IObgb2vIzAb5spLm6VA65h?mode=ems_copy_c")
        return

    # طريقة الشحن
    if text == "💰 طريقة الشحن":
        bot.send_message(chat_id, "اختر طريقة الشحن:", reply_markup=shipping_menu())
        return

    if text in ["📲 حساب سيرياتيل كاش", "💵 بدون سيرياتيل كاش"]:
        pending_orders[chat_id] = {"type": "شحن", "method": text}
        bot.send_message(chat_id, "أرسل رقم عملية التحويل:")
        return

    # استقبال رقم العملية والمبلغ
    if chat_id in pending_orders and "operation" not in pending_orders[chat_id] and pending_orders[chat_id]["type"] == "شحن":
        pending_orders[chat_id]["operation"] = text
        bot.send_message(chat_id, "أرسل المبلغ الذي حولته:")
        return

    if chat_id in pending_orders and "amount" not in pending_orders[chat_id] and pending_orders[chat_id]["type"] == "شحن":
        pending_orders[chat_id]["amount"] = text
        data = pending_orders[chat_id]
        markup = types.InlineKeyboardMarkup()
        markup.add(
            types.InlineKeyboardButton("✅ قبول", callback_data=f"accept_{chat_id}"),
            types.InlineKeyboardButton("❌ رفض", callback_data=f"reject_{chat_id}")
        )
        bot.send_message(ADMIN_ID,
            f"طلب شحن جديد:\n👤 @{message.from_user.username}\nطريقة: {data['method']}\nرقم العملية: {data['operation']}\nالمبلغ: {data['amount']}",
            reply_markup=markup
        )
        bot.send_message(chat_id, "✅ تم إرسال طلبك للإدمن للموافقة.")
        return

    # قوائم الألعاب
    if text == "💎 FREE FIRE":
        m = types.ReplyKeyboardMarkup(resize_keyboard=True)
        for p in freefire_packs.keys():
            m.add(p)
        m.add("⬅ العودة")
        bot.send_message(chat_id, "اختر فئة FREE FIRE:", reply_markup=m)
        return

    if text == "🎯 PUBG":
        m = types.ReplyKeyboardMarkup(resize_keyboard=True)
        for p in pubg_packs.keys():
            m.add(p)
        m.add("⬅ العودة")
        bot.send_message(chat_id, "اختر فئة PUBG:", reply_markup=m)
        return

    # اختيار الفئة
    if text in freefire_packs or text in pubg_packs:
        game = "FREE FIRE" if text in freefire_packs else "PUBG"
        price = apply_discount(freefire_packs.get(text, pubg_packs.get(text)))
        user_orders[chat_id] = {"game": game, "pack": text, "price": price}
        bot.send_message(chat_id, f"🎮 {text}\n💰 السعر بعد الخصم: {price} ل.س\n\nأرسل أيدي الحساب لتأكيد الطلب:")
        user_temp[chat_id] = {"step": "waiting_id"}
        return

    # إدخال الأيدي للشراء
    if chat_id in user_temp and user_temp[chat_id].get("step") == "waiting_id":
        user_temp[chat_id]["id"] = text
        order = user_orders.get(chat_id, {})
        pending_orders[chat_id] = {
            "type": "شراء",
            "game": order["game"],
            "pack": order["pack"],
            "price": order["price"],
            "id": text
        }
        # عرض معلومات الطلبية مع الأزرار الفرعية
        order_info = (
            "☠️    معلومات الطلبية    ☠️\n"
            f"• اللعبة: {order['game']}\n"
            f"• الحزمة: {order['pack']}\n"
            f"• السعر بالليرة: {order['price']}\n"
            f"• معرف اللاعب: {text}\n\n"
            "اختر احد الاجراءات الاتية:"
        )
        bot.send_message(chat_id, order_info, reply_markup=order_id_menu())
        user_temp.pop(chat_id, None)
        return

# التعامل مع أزرار الطلب الفرعية
@bot.callback_query_handler(func=lambda call: True)
def callback_handler(call):
    chat_id = call.message.chat.id
    data = call.data

    if data == "confirm_order":
        order = pending_orders.get(chat_id)
        if order:
            # إرسال طلب الشراء للإدمن مباشرة
            markup = types.InlineKeyboardMarkup()
            markup.add(
                types.InlineKeyboardButton("✅ قبول", callback_data=f"accept_{chat_id}"),
                types.InlineKeyboardButton("❌ رفض", callback_data=f"reject_{chat_id}")
            )
            bot.send_message(ADMIN_ID,
                f"طلب شراء جديد:\n👤 @{call.from_user.username}\n🎮 {order['game']}\n📦 {order['pack']}\n🆔 {order['id']}\n💰 السعر: {order['price']} ل.س",
                reply_markup=markup
            )
            bot.send_message(chat_id, "✅ تم إرسال طلبك للإدمن للموافقة.")
    elif data == "change_pack":
        bot.send_message(chat_id, "📌 الرجاء اختيار الفئة الجديدة من القائمة الرئيسية")
        main_menu(chat_id)
    elif data == "change_id":
        bot.send_message(chat_id, "📌 أرسل الأيدي الجديد:")
        user_temp[chat_id] = {"step": "waiting_id"}
    elif data == "cancel_order":
        pending_orders.pop(chat_id, None)
        bot.send_message(chat_id, "❌ تم إلغاء الطلب.")

    # أزرار قبول/رفض للإدمن
    elif data.startswith("accept_") or data.startswith("reject_"):
        admin_chat_id = int(data.split("_")[1])
        if admin_chat_id not in pending_orders:
            bot.answer_callback_query(call.id, "هذا الطلب غير موجود")
            return
        order = pending_orders[admin_chat_id]
        if data.startswith("accept_"):
            if order["type"] == "شحن":
                bot.send_message(admin_chat_id, f"✅ تمت إضافة {order['amount']} ل.س إلى رصيدك.")
            elif order["type"] == "شراء":
                bot.send_message(admin_chat_id, f"✅ تم تأكيد طلب شراء {order['pack']} للعبة {order['game']}.")
            bot.answer_callback_query(call.id, "تمت الموافقة")
        else:
            bot.send_message(admin_chat_id, "❌ تم رفض طلبك.")
            bot.answer_callback_query(call.id, "تم الرفض")
        pending_orders.pop(admin_chat_id, None)

print("🚀 البوت جاهز للعمل!")
bot.polling(none_stop=True)
