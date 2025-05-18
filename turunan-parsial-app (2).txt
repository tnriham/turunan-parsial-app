import streamlit as st
import sympy as sp
import numpy as np
import matplotlib.pyplot as plt

# Konfigurasi awal
st.set_page_config(page_title="Studi Kasus Turunan Parsial")
st.title("📘 Studi Kasus Turunan Parsial")
st.markdown("## Optimasi Biaya Produksi Baterai Kendaraan Listrik")

# Latar belakang
st.markdown("""
Perusahaan manufaktur kendaraan listrik (EV) seperti **Tesla**, **BYD**, dan **CATL** menghadapi tantangan dalam menurunkan biaya produksi baterai — komponen paling mahal dalam EV. Mereka memproduksi dua jenis modul baterai:

- `x` : Modul baterai **standar**
- `y` : Modul baterai **performa tinggi**

Biaya produksi total tergantung pada jumlah masing-masing modul yang diproduksi.
""")

st.markdown("---")
st.header("📌 1. Fungsi Biaya Produksi")
default_fungsi = "5*x**2 + 4*x*y + 8*y**2 + 300*x + 500*y + 10000"
fungsi_str = st.text_input("Masukkan fungsi biaya produksi total C(x, y):", default_fungsi)

x, y = sp.symbols('x y')

try:
    # Simbolik dan turunan
    f = sp.sympify(fungsi_str)
    fx = sp.diff(f, x)
    fy = sp.diff(f, y)

    st.latex(f"C(x, y) = {sp.latex(f)}")
    st.latex(f"\\frac{{\\partial C}}{{\\partial x}} = {sp.latex(fx)}")
    st.latex(f"\\frac{{\\partial C}}{{\\partial y}} = {sp.latex(fy)}")

    st.markdown("---")
    st.header("📊 2. Evaluasi Nilai di Titik Produksi")

    st.write("Misalkan perusahaan ingin memproduksi:")
    col1, col2 = st.columns(2)
    with col1:
        x0 = st.number_input("Jumlah modul standar (x₀):", value=50.0)
    with col2:
        y0 = st.number_input("Jumlah modul performa tinggi (y₀):", value=30.0)

    f_val = f.subs({x: x0, y: y0})
    fx_val = fx.subs({x: x0, y: y0})
    fy_val = fy.subs({x: x0, y: y0})

    # Hitung dalam rupiah: fx * 17.000 dan fy * 16.000
    biaya_x = fx_val * 17000
    biaya_y = fy_val * 16000

    st.success(f"💰 Total biaya produksi pada titik ({x0}, {y0}): {f_val} dolar")
    st.info(f"📈 Gradien di titik tersebut: (∂C/∂x, ∂C/∂y) = ({fx_val}, {fy_val})")

    st.markdown(f"""
    - ∂C/∂x = {fx_val} ⟶ Rp {biaya_x:,.0f}
    - ∂C/∂y = {fy_val} ⟶ Rp {biaya_y:,.0f}
    """)

    st.markdown("---")
    st.header("📈 3. Visualisasi Permukaan & Bidang Singgung")

    x_vals = np.linspace(x0 - 20, x0 + 20, 50)
    y_vals = np.linspace(y0 - 20, y0 + 20, 50)
    X, Y = np.meshgrid(x_vals, y_vals)
    Z = sp.lambdify((x, y), f, 'numpy')(X, Y)
    Z_tangent = float(f_val) + float(fx_val)*(X - x0) + float(fy_val)*(Y - y0)

    fig = plt.figure(figsize=(10, 6))
    ax = fig.add_subplot(111, projection='3d')
    ax.plot_surface(X, Y, Z, cmap='viridis', alpha=0.8)
    ax.plot_surface(X, Y, Z_tangent, color='orange', alpha=0.4)
    ax.set_title("Permukaan Fungsi Biaya dan Bidang Singgung")
    ax.set_xlabel("x (Modul Standar)")
    ax.set_ylabel("y (Modul Performa Tinggi)")
    ax.set_zlabel("C(x, y)")
    st.pyplot(fig)

except Exception as e:
    st.error(f"Terjadi kesalahan dalam pengolahan fungsi: {e}")
