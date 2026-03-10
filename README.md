from fastapi import FastAPI, BackgroundTasks
import yfinance as yf
import asyncio

app = FastAPI()

# محاكي لقاعدة بيانات التنبيهات
alerts_db = {"NVDA": 900.0} 

def check_price_logic():
    print("🔔 محرك الرصد يعمل في الخلفية...")
    for symbol, target in alerts_db.items():
        ticker = yf.Ticker(symbol)
        price = ticker.fast_info['last_price']
        if price >= target:
            print(f"✅ تنبيه: {symbol} وصل إلى {price}$!")

@app.on_event("startup")
async def startup_event():
    # يبدأ المحرك عند تشغيل السيرفر
    asyncio.create_task(run_periodic_check())

async def run_periodic_check():
    while True:
        check_price_logic()
        await asyncio.sleep(60) # يفحص كل دقيقة

@app.get("/status")
async def get_status():
    return {"message": "System Operational", "monitored_stocks": list(alerts_db.keys())}
