import random
import time
# В реальной жизни здесь использовались бы модули Pulseq
# import pypulseq as mr 

# --- Имитация основных функций Pulseq ---

def create_rf_pulse(flip_angle, duration_ms):
    """Имитирует создание РЧ-импульса (Радиочастотного)."""
    return {"type": "RF", "angle": f"{flip_angle}°", "duration": f"{duration_ms} ms"}

def create_gradient(axis, amplitude_mT_m, duration_us):
    """Имитирует создание градиента (Линейное изменение поля)."""
    return {"type": "GRAD", "axis": axis, "amplitude": f"{amplitude_mT_m} mT/m", "duration": f"{duration_us} us"}

def create_adc(dwell_time_us, num_samples):
    """Имитирует создание окна сбора данных (Аналого-Цифровой Преобразователь)."""
    return {"type": "ADC", "dwell": f"{dwell_time_us} us", "samples": num_samples}

# --- Основная функция: Программирование последовательности ---

def program_gradient_echo_sequence(
        TR_ms=100, 
        TE_ms=5, 
        flip_angle=random.randint(5, 90)
    ):
    """
    Программирует простую 2D последовательность Градиентное Эхо (GRE).
    GRE - это самая быстрая и базовая последовательность.
    """
    
    # 1. Основные параметры (рандомно сгенерированные для примера)
    BW_Hz = 100000  # Ширина полосы пропускания (Bandwidth)
    FOV_mm = 256    # Поле обзора (Field of View)
    matrix = 256    # Размер матрицы (Matrix Size)
    
    # Рассчеты для градиентов и сбора данных
    readout_duration_us = int(matrix * (1000000 / BW_Hz)) # Время считывания (в мкс)
    time_to_center_us = int(readout_duration_us / 2)
    
    # --- Инициализация последовательности ---
    sequence = []
    
    # --- 1. RF-Импульс возбуждения ---
    # Угол наклона определяет контраст (Т1-зависимость)
    rf_pulse = create_rf_pulse(flip_angle, duration_ms=1.0)
    sequence.append(rf_pulse)
    
    # --- 2. Градиент фазового кодирования (Pre-Phasing Gradient) ---
    # Градиент, который задает первую точку в k-пространстве (Phase Encoding)
    phase_encode_grad = create_gradient('Y', amplitude_mT_m=random.uniform(-5, 5), duration_us=500)
    sequence.append(phase_encode_grad)
    
    # --- 3. Градиент считывания (Readout Dephasing) ---
    # Градиент, который дефазирует сигнал перед считыванием
    read_dephase_grad = create_gradient('X', amplitude_mT_m=random.uniform(-10, -5), duration_us=1000)
    sequence.append(read_dephase_grad)

    # --- 4. Задержка до TE (Time to Echo) ---
    # Время между RF-импульсом и центром сбора данных (TE)
    delay_to_te = {"type": "DELAY", "duration": f"{TE_ms - 2} ms"}
    sequence.append(delay_to_te)
    
    # --- 5. Сбор данных (ADC) и Градиент считывания ---
    # ADC находится в центре градиента считывания
    readout_grad = create_gradient('X', amplitude_mT_m=random.uniform(5, 10), duration_us=readout_duration_us)
    adc_acquisition = create_adc(dwell_time_us=int(1000000 / BW_Hz), num_samples=matrix)
    
    sequence.append(readout_grad)
    sequence.append(adc_acquisition)

    # --- 6. Задержка до TR (Time to Repetition) ---
    # TR - полное время цикла
    delay_to_tr = {"type": "DELAY", "duration": f"{TR_ms - TE_ms} ms"}
    sequence.append(delay_to_tr)
    
    return sequence, TR_ms, flip_angle

# --- Выполнение ---

# Случайные параметры
TR_RANDOM = random.randint(50, 200)
TE_RANDOM = random.randint(3, 15)
ANGLE_RANDOM = random.randint(5, 45) # Для GRE обычно низкий угол

pulse_sequence, TR, FA = program_gradient_echo_sequence(TR_ms=TR_RANDOM, TE_ms=TE_RANDOM, flip_angle=ANGLE_RANDOM)

print("--- 🩺 ИМИТАЦИЯ ПРОГРАММЫ МРТ-ПОСЛЕДОВАТЕЛЬНОСТИ (Gradient Echo) ---")
print(f"**Параметры протокола:** TR={TR} ms, TE={TE_RANDOM} ms, Угол наклона={FA}°")
print("-" * 75)

# Вывод первых нескольких шагов последовательности
for step in pulse_sequence:
    if step["type"] == "RF":
        print(f"| 📡 RF-Pulse: Возбуждение - {step['angle']} |")
    elif step["type"] == "GRAD":
        print(f"| 📈 Gradient ({step['axis']}): Амплитуда {step['amplitude']}, Длительность {step['duration']} |")
    elif step["type"] == "ADC":
        print(f"| 🖥️ ADC: Сбор данных - {step['samples']} точек |")
    elif step["type"] == "DELAY":
        print(f"| ⏳ DELAY: Ожидание - {step['duration']} |")

print("-" * 75)
print("Это один цикл TR. В реальном сканере цикл повторяется многократно для сбора K-пространства.")
