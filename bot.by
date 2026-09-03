import time
import requests
import pandas as pd
import yfinance as yf

TOKEN = "8886031214:AAFn_sFsqUdpzfcgX62f07RQEFldO3Jvumy0"
CHAT_ID = "5040690530"
SYMBOL = "GC=F"  # رمز الذهب (أو استبدله بأي زوج تريده مثل EURUSD=X)

def send_telegram_message(text):
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    payload = {"chat_id": CHAT_ID, "text": text, "parse_mode": "Markdown"}
    try:
        return requests.post(url, json=payload, timeout=10).json()
    except Exception as e:
        print("خطأ في الاتصال:", e)

def check_market():
    print(f"[{time.strftime('%H:%M:%S')}] جاري فحص السوق والبحث عن إشارات...")
    try:
        df = yf.download(SYMBOL, period="5d", interval="1h", progress=False)
        if df.empty:
            print("لم يتم استرجاع البيانات.")
            return
        if isinstance(df.columns, pd.MultiIndex):
            df.columns = df.columns.get_level_values(0)

        # حساب مؤشرات الهيكل والسيولة المماثلة لمنطق مؤشرك
        df['ATR'] = (df['High'] - df['Low']).rolling(14).mean()
        df['Swing_High'] = df['High'].rolling(10).max()
        df['Swing_Low'] = df['Low'].rolling(10).min()
        
        last = df.iloc[-2]
        prev = df.iloc[-3]
        price = df.iloc[-1]['Close']

        # شروط الإشارات بناءً على القمم والقيعان ومناطق السيولة
        buy_signal = (prev['Close'] <= prev['Swing_Low']) and (last['Close'] > last['Swing_Low'])
        sell_signal = (prev['Close'] >= prev['Swing_High']) and (last['Close'] < last['Swing_High'])
        rebound_zone = abs(price - last['Swing_Low']) <= last['ATR'] * 0.8

        if buy_signal:
            send_telegram_message(f"🟢 *صفقة شراء جديدة (BUY)*\n- الأصل: `{SYMBOL}`\n- السعر: `{price:.2f}`\n- الحالة: ارتداد إيجابي من القاع الهيكلي.")
            print("تم إرسال تنبيه شراء!")
        elif sell_signal:
            send_telegram_message(f"🔴 *صفقة بيع جديدة (SELL)*\n- الأصل: `{SYMBOL}`\n- السعر: `{price:.2f}`\n- الحالة: كسر سلبي من القمة الهيكلية.")
            print("تم إرسال تنبيه بيع!")
        elif rebound_zone:
            send_telegram_message(f"🎯 *منطقة ارتداد ودخول شراء محتمل*\n- الأصل: `{SYMBOL}`\n- السعر الحالي: `{price:.2f}`\n- الحالة: السعر اقترب من منطقة طلب/سيولة.")
            print("تم إرسال تنبيه منطقة الارتداد!")
        else:
            print("لا توجد إشارة جديدة حالياً، السوق تحت المراقبة.")
    except Exception as e:
        print("حدث خطأ أثناء فحص البيانات:", e)

send_telegram_message("🤖 تم تشغيل نظام المراقبة الذكي لمؤشر الفخامة بنجاح، ويتم فحص السوق الآن دورياً.")

# حلقة تكرارية تعمل في الخلفية لفحص السوق كل 5 دقائق (300 ثانية)
while True:
    check_market()
    time.sleep(300)
