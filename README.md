import numpy as np
import matplotlib.pyplot as plt
import random

# --- Настройки генерации данных ---
SAMPLE_SIZE = 10000  # Количество точек данных
MU = random.uniform(50, 150)  # Среднее значение (Miu - μ)
SIGMA = random.uniform(10, 30) # Стандартное отклонение (Sigma - σ)

def generate_and_analyze_data(size, mu, sigma):
    """
    Генерирует случайные данные по нормальному распределению и вычисляет метрики.
    """
    
    # 1. Генерация данных (используем numpy для эффективности)
    # np.random.normal(loc=среднее, scale=станд.откл., size=количество)
    data = np.random.normal(loc=mu, scale=sigma, size=size)
    
    # 2. Вычисление статистических метрик
    calculated_mean = np.mean(data)
    calculated_std = np.std(data)
    
    # 3. Визуализация
    plt.figure(figsize=(10, 6))
    
    # Гистограмма данных
    plt.hist(data, bins=50, density=True, alpha=0.6, color='skyblue', label='Гистограмма данных')
    
    # Построение теоретической кривой нормального распределения
    # 

[Image of Normal distribution curve with mean and standard deviation]

    xmin, xmax = plt.xlim()
    x = np.linspace(xmin, xmax, 100)
    p = (1 / (sigma * np.sqrt(2 * np.pi))) * np.exp(-0.5 * ((x - mu) / sigma)**2)
    plt.plot(x, p, 'r', linewidth=2, label=f'Теоретическая кривая (μ={mu:.2f}, σ={sigma:.2f})')
    
    # Добавление вертикальных линий для среднего и отклонений
    plt.axvline(calculated_mean, color='green', linestyle='dashed', linewidth=2, label=f'Среднее (расч.)')
    plt.axvline(calculated_mean + calculated_std, color='gray', linestyle='dotted', linewidth=1)
    plt.axvline(calculated_mean - calculated_std, color='gray', linestyle='dotted', linewidth=1)
    
    plt.title(f'Генерация данных по нормальному распределению (N={size})')
    plt.xlabel('Значение')
    plt.ylabel('Плотность вероятности')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.show()

    return data, calculated_mean, calculated_std

# --- Главная часть программы ---

print("--- 📊 АНАЛИЗ ДАННЫХ ПО НОРМАЛЬНОМУ РАСПРЕДЕЛЕНИЮ ---")
print(f"Заданные параметры: Среднее (μ)={MU:.2f}, Стандартное отклонение (σ)={SIGMA:.2f}")
print(f"Размер выборки: {SAMPLE_SIZE}")
print("-" * 60)

# Запуск генерации и анализа
generated_data, calculated_mean, calculated_std = generate_and_analyze_data(SAMPLE_SIZE, MU, SIGMA)

print(f"\nРезультаты анализа:")
print(f"Расчетное среднее (Mean): {calculated_mean:.4f}")
print(f"Расчетное ст. отклонение (Std Dev): {calculated_std:.4f}")
print("Визуализация отображена в отдельном окне Matplotlib.")
