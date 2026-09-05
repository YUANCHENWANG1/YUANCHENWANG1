# 文件名: app_mobile.py (安全加固版)
import streamlit as st
import pandas as pd
import numpy as np
from datetime import datetime

st.set_page_config(page_title="期货智析·手机版", layout="centered")

# 容错CSS
st.markdown("""
<style>
    .reportview-container .main .block-container { max-width: 600px; padding: 1rem 0.8rem; margin: 0 auto; }
    .css-1y4p8pa { font-size: 1rem !important; }
    .css-1xarl3l { font-size: 1.6rem !important; }
    .stSelectbox, .stButton button { font-size: 1.2rem !important; padding: 0.5rem !important; }
    h1 { font-size: 2rem !important; }
</style>
""", unsafe_allow_html=True)

st.title("📊 期货智析")
st.caption(f"📱 手机专属 | {datetime.now().strftime('%m-%d %H:%M')}")

# ---------- 安全的数据加载 ----------
@st.cache_data
def load_data():
    try:
        raw = {
            "RB": {"name": "螺纹钢", "spot": 3750, "near": 3720, "far": 3650, "inventory": 580, "profit": 185, "warehouse": 85600, "reg": 1200, "cancel": 450},
            "I":  {"name": "铁矿石", "spot": 860, "near": 850, "far": 810, "inventory": 13500, "profit": 92, "warehouse": 2300, "reg": 0, "cancel": 200},
            "SA": {"name": "纯碱", "spot": 2250, "near": 2200, "far": 2100, "inventory": 320, "profit": 260, "warehouse": 1500, "reg": 500, "cancel": 100},
            "CU": {"name": "铜", "spot": 69000, "near": 68800, "far": 68500, "inventory": 280, "profit": 800, "warehouse": 42000, "reg": 800, "cancel": 300},
            "AU": {"name": "黄金", "spot": 478, "near": 476, "far": 475, "inventory": 10, "profit": 15, "warehouse": 3500, "reg": 50, "cancel": 80},
        }
        df = pd.DataFrame(raw).T.reset_index().rename(columns={"index": "code"})
        # 计算衍生字段
        df["spread"] = df["near"] - df["far"]
        df["structure"] = df["spread"].apply(lambda x: "Back" if x > 0 else "Contango")
        return df, raw
    except Exception as e:
        st.error(f"数据加载异常，请检查格式: {e}")
        return pd.DataFrame(), {}

df_raw, data = load_data()

# 如果数据加载失败，显示占位信息
if df_raw.empty:
    st.stop()

# ---------- 1. Back结构 ----------
st.subheader("🔥 热门品种榜")
try:
    df_top = df_raw.sort_values("spread", ascending=False)
    show_cols = ["code", "name", "near", "far", "spread", "structure"]
    df_show = df_top[show_cols].copy()
    df_show.columns = ["代码", "品种", "近月", "远月", "价差", "结构"]
    st.dataframe(df_show, use_container_width=True, hide_index=True, height=250)
    best = df_top.iloc[0]
    st.metric("🏆 当前最热", f"{best['name']} ({best['code']})", delta=f"价差 {best['spread']} 元")
except Exception as e:
    st.warning("暂无Back数据")
st.divider()

# ---------- 2. 基差 ----------
st.subheader("💹 基差分析")
try:
    codes = df_raw["code"].tolist()
    symbol_basis = st.selectbox("选择品种", codes, key="basis", format_func=lambda x: f"{x} - {data.get(x, {}).get('name', '')}")
    row = df_raw[df_raw["code"] == symbol_basis].iloc[0]
    basis_val = row["spot"] - row["near"]
    basis_rate = (basis_val / row["spot"]) * 100
    c1, c2 = st.columns(2)
    c1.metric("现货", row["spot"])
    c2.metric("期货", row["near"])
    c3, c4 = st.columns(2)
    c3.metric("基差", f"{basis_val:.1f}")
    c4.metric("基差率", f"{basis_rate:.2f}%")
except Exception as e:
    st.warning("基差数据异常")
st.divider()

# ---------- 3. 利润库存 ----------
st.subheader("⚙️ 利润 & 库存")
try:尝试：
    symbol_profit = st.selectbox("选择品种", codes, key="profit", format_func=lambda x: f"{x} - {data.get(x, {}).get('name', '')}")
    row_p = df_raw[df_raw["code""代码"] == symbol_profit].iloc[0]
    col1, col2 = st.columns(2)
    col1.metric("📦 库存(万吨)", f"{row_p['inventory']:,}")
    col2.metric("💰 利润(元)", f"{row_p['profit']}")
    # 迷你图
    st.bar_chart(pd.DataFrame({"指标": ["库存", "利润"], "数值": [row_p['inventory']/10, row_p['profit']/10]}).set_index("指标"), height=150)
except Exception as e:
    st.warning("利润数据异常")
st.divider()

# ---------- 4. 仓单 ----------
st.subheader("🏢 仓单动态")
try:尝试：
    symbol_wh = st.selectbox("选择品种", codes, key="wh", format_func=lambda x: f"{x} - {data.get(x, {}).get('name', '')}")
    row_w = df_raw[df_raw["code""代码"] == symbol_wh].iloc[0]
    net_change = row_w["reg"] - row_w["cancel"“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”“取消”]
    cols = st.columns(2)
    cols[0].metric("📋 总仓单", f"{row_w['warehouse']:,}")
    cols[1].metric("📤 新注册", f"+{row_w['reg']:,}")
    cols2 = st.columns(2)
    cols2[0].metric("📥 新注销", f"-{row_w['cancel']:,}")
    cols2[1].metric("🔄 净变化", f"{net_change:+d}")
except Exception as e:
    st.warning("仓单数据异常")
st.divider()

st.success("✅ 页面加载成功 (若数据有误，请查看终端红色报错)")
st.caption("💡 数据为模拟示例")
