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
import React, { useEffect, useState } from 'react';

export default function Dashboard() {
  const [stock, setStock] = useState({ symbol: 'NVDA', price: 'Loading...' });

  useEffect(() => {
    fetch('http://127.0.0.1:8000/price/NVDA')
      .then(res => res.json())
      .then(data => setStock(data));
  }, []);

  return (
    <div style={{ padding: '50px', background: '#111', color: '#fff', textAlign: 'center' }}>
      <h1>Nasdaq Monitor Pro</h1>
      <div style={{ fontSize: '40px', color: '#00ff00' }}>
        {stock.symbol}: ${stock.price}
      </div>
    </div>
  );
}
uvicorn app.main:app --reload
