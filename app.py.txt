import streamlit as st
import pandas as pd
import plotly.express as px

st.set_page_config(page_title="Folk Healer Dashboard", page_icon="🌿", layout="wide")

# CSS ตกแต่ง
st.markdown("""
<style>
    .metric-card {background-color: #f0f2f6; border-radius: 10px; padding: 15px; text-align: center;}
</style>
""", unsafe_allow_html=True)

st.title("🌿 ระบบสารสนเทศภูมิปัญญาหมอพื้นบ้าน")
st.markdown("---")

# --- ส่วนอัปโหลดไฟล์ ---
st.sidebar.header("📂 นำเข้าข้อมูล")
uploaded_file = st.sidebar.file_uploader("อัปโหลดไฟล์ Excel ที่นี่", type=["xlsx", "xls"])

if uploaded_file is not None:
    try:
        df = pd.read_excel(uploaded_file)
        df.columns = df.columns.str.strip() # ลบช่องว่างชื่อคอลัมน์

        # --- ส่วนกรองข้อมูล ---
        st.sidebar.subheader("🔍 ตัวกรอง")
        
        # ตรวจสอบว่ามีคอลัมน์ 'อำเภอ' ไหม
        if 'อำเภอ' in df.columns:
            all_amphoe = ["ทั้งหมด"] + sorted(df['อำเภอ'].astype(str).unique().tolist())
            selected_amphoe = st.sidebar.selectbox("เลือกอำเภอ:", all_amphoe)
            if selected_amphoe != "ทั้งหมด":
                df = df[df['อำเภอ'] == selected_amphoe]

        # --- แสดงผล KPI ---
        col1, col2, col3, col4 = st.columns(4)
        col1.metric("จำนวนหมอ (คน)", len(df))
        
        if 'อายุ' in df.columns:
            col2.metric("อายุเฉลี่ย", f"{df['อายุ'].mean():.1f}")
        else:
            col2.metric("อายุเฉลี่ย", "-")
            
        if 'ความชำนาญโรค' in df.columns:
            col3.metric("ความชำนาญสูงสุด", df['ความชำนาญโรค'].mode()[0] if not df.empty else "-")
        else:
            col3.metric("ความชำนาญสูงสุด", "-")
            
        if 'อำเภอ' in df.columns:
            col4.metric("พื้นที่ (อำเภอ)", df['อำเภอ'].nunique())
        else:
            col4.metric("พื้นที่", "-")

        st.markdown("---")

        # --- แสดงกราฟ ---
        c1, c2 = st.columns(2)
        
        with c1:
            if 'ประเภทของหมอพื้นบ้าน' in df.columns:
                st.subheader("🗂️ สัดส่วนประเภทหมอ")
                fig_pie = px.pie(df, names='ประเภทของหมอพื้นบ้าน', hole=0.4)
                st.plotly_chart(fig_pie, use_container_width=True)
        
        with c2:
            if 'อำเภอ' in df.columns:
                st.subheader("📍 การกระจายตัวตามอำเภอ")
                loc_counts = df['อำเภอ'].value_counts().reset_index()
                loc_counts.columns = ['อำเภอ', 'จำนวน']
                fig_bar = px.bar(loc_counts, x='อำเภอ', y='จำนวน', color='จำนวน')
                st.plotly_chart(fig_bar, use_container_width=True)

        # --- ตารางข้อมูล ---
        st.subheader("📝 รายละเอียดข้อมูล")
        st.dataframe(df, use_container_width=True)

    except Exception as e:
        st.error(f"ไฟล์มีปัญหา หรือชื่อคอลัมน์ไม่ตรง: {e}")
else:
    st.info("👋 กรุณาอัปโหลดไฟล์ Excel ทางแถบซ้ายมือ เพื่อเริ่มใช้งาน")
    st.write("ระบบนี้จะไม่บันทึกข้อมูลของคุณลง Server ปลอดภัยหายห่วงครับ")