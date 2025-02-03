import os
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import lasio
import streamlit as st

# Функция для поиска информации о горизонте/ярусе в LAS-файле
def find_horizon_info(las):
    possible_keys = ['ZONE', 'HORIZON', 'FORMATION', 'HOR', 'ZON']
    for key in possible_keys:
        if key in las.params:
            value = las.params.get(key).value
            if value and value.strip():
                return clean_horizon_value(value)
    return 'Неопределено'

# Функция для очистки значения горизонта от лишних кавычек и пробелов
def clean_horizon_value(value):
    if value is None or pd.isna(value):
        return 'Неопределено'
    cleaned_value = value.strip().strip('"').strip("'")
    return cleaned_value if cleaned_value != '' else 'Неопределено'

# Загрузка всех LAS файлов из указанной папки
def load_las_files(folder_path):
    files = [f for f in os.listdir(folder_path) if f.endswith('.las')]
    data_frames = []
    for file in files:
        file_path = os.path.join(folder_path, file)
        las = lasio.read(file_path)
        df = las.df().reset_index()
        df.columns = [col.strip().upper() for col in df.columns]
        horizon_zone = find_horizon_info(las)
        df['Горизонт_ярус'] = str(horizon_zone)
        if 'КОЛЛЕКТОР' not in df.columns:
            df['КОЛЛЕКТОР'] = 'Неопределено'
        if 'ЛИТОЛОГИЯ' not in df.columns:
            df['ЛИТОЛОГИЯ'] = 'Неопределено'
        data_frames.append(df)
    combined_df = pd.concat(data_frames, ignore_index=True)
    return combined_df

# Построение распределения частот IK и BK
def plot_frequency_distribution(df, shift_ik=0, shift_bk=0):
    if 'IK' not in df.columns or 'BK' not in df.columns:
        st.write("Столбцы 'IK' или 'BK' отсутствуют в данных.")
        return
    ik_shifted = df['IK'] + shift_ik
    bk_shifted = df['BK'] + shift_bk
    plt.figure(figsize=(10, 6))
    sns.kdeplot(ik_shifted, label='IK', color=(0 / 255, 180 / 255, 115 / 255))
    sns.kdeplot(bk_shifted, label='BK', color=(153 / 255, 153 / 255, 153 / 255))
    plt.title('Распределение частот IK и BK', fontsize=12)
    plt.xlabel('Значения', fontsize=12)
    plt.ylabel('Частота', fontsize=12)
    plt.legend(fontsize=10)
    st.pyplot(plt)

# Интерактивное построение графика
def interactive_plot(df):
    st.sidebar.header("Настройки фильтрации")
    horizons = ['All'] + list(df['ГОРИЗОНТ_ЯРУС'].dropna().astype(str).unique())
    horizon = st.sidebar.multiselect("Выберите горизонт:", horizons, default=['All'])
    collectors = ['All'] + list(df['КОЛЛЕКТОР'].dropna().unique())
    collector = st.sidebar.selectbox("Выберите коллектор:", collectors)
    lithologies = ['All'] + list(df['ЛИТОЛОГИЯ'].dropna().unique())
    lithology = st.sidebar.selectbox("Выберите литологию:", lithologies)

    # Смещение IK и BK
    shift_ik = st.sidebar.slider("Смещение IK:", -10.0, 10.0, 0.0, 0.1)
    shift_bk = st.sidebar.slider("Смещение BK:", -10.0, 10.0, 0.0, 0.1)

    # Фильтры по IK и BK
    ik_min = st.sidebar.slider("Минимальное значение IK:", float(df['IK'].min()), float(df['IK'].max()), float(df['IK'].min()))
    ik_max = st.sidebar.slider("Максимальное значение IK:", float(df['IK'].min()), float(df['IK'].max()), float(df['IK'].max()))
    bk_min = st.sidebar.slider("Минимальное значение BK:", float(df['BK'].min()), float(df['BK'].max()), float(df['BK'].min()))
    bk_max = st.sidebar.slider("Максимальное значение BK:", float(df['BK'].min()), float(df['BK'].max()), float(df['BK'].max()))

    filtered_df = filter_data(df, horizon, collector, lithology, ik_min, ik_max, bk_min, bk_max)
    if filtered_df.empty:
        st.write("Нет данных после применения фильтра.")
    else:
        plot_frequency_distribution(filtered_df, shift_ik, shift_bk)

# Фильтрация данных
def filter_data(df, horizon=None, collector=None, lithology=None, ik_min=None, ik_max=None, bk_min=None, bk_max=None):
    filtered_df = df.copy()
    if horizon and horizon != ['All']:
        filtered_df = filtered_df[filtered_df['ГОРИЗОНТ_ЯРУС'].isin(horizon)]
    if collector and collector != 'All':
        filtered_df = filtered_df[filtered_df['КОЛЛЕКТОР'] == collector]
    if lithology and lithology != 'All':
        filtered_df = filtered_df[filtered_df['ЛИТОЛОГИЯ'] == lithology]
    if ik_min is not None and ik_max is not None and 'IK' in df.columns:
        filtered_df = filtered_df[(filtered_df['IK'] >= ik_min) & (filtered_df['IK'] <= ik_max)]
    if bk_min is not None and bk_max is not None and 'BK' in df.columns:
        filtered_df = filtered_df[(filtered_df['BK'] >= bk_min) & (filtered_df['BK'] <= bk_max)]
    return filtered_df

# Основная функция
def main():
    st.title("Анализ данных LAS-файлов")
    folder_path = st.text_input("Введите путь к папке с LAS файлами:")
    if folder_path and os.path.exists(folder_path):
        try:
            df = load_las_files(folder_path)
            if df.empty:
                st.write("Не удалось загрузить данные из LAS файлов.")
            else:
                st.write("Первые 5 значений для каждой колонки:")
                st.write(df.head())
                interactive_plot(df)
        except Exception as e:
            st.write(f"Произошла ошибка: {e}")
    else:
        st.write("Пожалуйста, введите корректный путь к папке.")

if __name__ == "__main__":
    main()
