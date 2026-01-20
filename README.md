# trader-assistentebackend/main.py
from fastapi import FastAPI, Header, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import pandas as pd
import requests
import jwt
from ta.momentum import RSIIndicator
from ta.trend import EMAIndicator

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

def buscar_candles(symbol, interval):
    url = "https://api.binance.com/api/v3/klines"
    data = requests.get(url, params={
        "symbol": symbol,
        "interval": interval,
        "limit": 120
    }).json()

    df = pd.DataFrame(data, columns=[
        't','o','h','l','c','v','ct','q','n','tb','tq','i'
    ])
    df['c'] = df['c'].astype(float)
    return df

@app.get("/analisar")
def analisar(symbol: str = "BTCUSDT", interval: str = "1m"):
    df = buscar_candles(symbol, interval)

    preco = df['c'].iloc[-1]
    ema20 = EMAIndicator(df['c'], 20).ema_indicator().iloc[-1]
    rsi = RSIIndicator(df['c'], 14).rsi().iloc[-1]

    if rsi < 25:
        decisao = "CALL"
    elif rsi > 75:
        decisao = "PUT"
    else:
        decisao = "NÃO OPERAR"

    return {
        "decisao": decisao,
        "rsi": round(rsi, 2),
        "preco": round(preco, 2)
    }
