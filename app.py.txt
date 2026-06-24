import streamlit as st
import pandas as pd
import plotly.express as px

# Page Config
st.set_page_config(page_title="Inventory Analytics", layout="wide")

# Title & Description
st.title("📦 Product Inventory Analysis System")
st.markdown("Automated system to track stock levels, detect anomalies, and analyze performance.")

# 1. Load Data
@st.cache_data
def load_data():
    df = pd.read_csv('inventory_data.csv')
    return df

df = load_data()

# 2. Logic: Structural Analysis & Anomaly Detection
# Condition-based logic for Status
df['Status'] = df.apply(lambda x: '🚨 CRITICAL' if x['Stock_Level'] < (x['Threshold'] * 0.4) 
                        else ('⚠️ Low Stock' if x['Stock_Level'] <= x['Threshold'] else '✅ Optimal'), axis=1)

# 3. Sidebar Filters
st.sidebar.header("Filter Options")
category_filter = st.sidebar.multiselect("Select Category", options=df['Category'].unique(), default=df['Category'].unique())
filtered_df = df[df['Category'].isin(category_filter)]

# 4. Dashboard Metrics
col1, col2, col3 = st.columns(3)
col1.metric("Total Products", len(filtered_df))
col2.metric("Critical Alerts", len(filtered_df[filtered_df['Status'] == '🚨 CRITICAL']))
col3.metric("Total Inventory Value", f"${(filtered_df['Stock_Level'] * filtered_df['Unit_Price']).sum():,.2f}")

# 5. Visual Analysis (Performance Profiles)
st.subheader("Performance Profile: Stock vs Threshold")
fig = px.bar(filtered_df, x='Product_Name', y=['Stock_Level', 'Threshold'], 
             barmode='group', title="Current Stock Levels vs. Safety Thresholds")
st.plotly_chart(fig, use_container_width=True)

# 6. Data Table with Anomalies
st.subheader("Detailed Inventory Audit Log")
st.dataframe(filtered_df.style.highlight_between(left=0, right=5, subset=['Stock_Level'], color='#ffcccc'), use_container_width=True)

# 7. Export Functionality
csv = filtered_df.to_csv(index=False).encode('utf-8')
st.download_button(label="📥 Download Audit Summary", data=csv, file_name='inventory_report.csv', mime='text/csv')