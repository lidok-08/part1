import requests
# import json
# import pprint
from tkinter import *
from tkinter import ttk
from tkinter import messagebox as mb

def update_bs_label(event):
    code = base_combobox.get()
    name = currencies.get(code, "Неизвестная валюта")
    bs_label.config(text=name)

def update_bs2_label(event):
    code = base2_combobox.get()
    name = currencies.get(code, "Неизвестная валюта")
    bs2_label.config(text=name)

def update_currency_label(event):
    code = target_combobox.get()
    name = currencies.get(code, "Неизвестная валюта")
    currency_label.config(text=name)

def exchange():
    target_code = target_combobox.get().strip()
    base_code = base_combobox.get().strip()
    base2_code = base2_combobox.get().strip()

    if not target_code or not base_code or not base2_code:
        mb.showwarning("Внимание", "Выберите все валюты")
        return

    try:
        # Получаем курсы для первой базовой валюты
        response1 = requests.get(f"https://open.er-api.com/v6/latest/{base_code}", timeout=10)
        response1.raise_for_status()
        data1 = response1.json()

        # Получаем курсы для второй базовой валюты
        response2 = requests.get(f"https://open.er-api.com/v6/latest/{base2_code}", timeout=10)
        response2.raise_for_status()
        data2 = response2.json()

        # Проверка структуры ответа
        if "rates" not in data1 or "rates" not in data2:
            mb.showerror("Ошибка", "API вернул неожиданный формат данных")
            return

        rates1 = data1["rates"]
        rates2 = data2["rates"]

        if target_code not in rates1:
            mb.showerror("Ошибка", f"Валюта {target_code} не найдена в курсе для {base_code}")
            return
        if target_code not in rates2:
            mb.showerror("Ошибка", f"Валюта {target_code} не найдена в курсе для {base2_code}")
            return

        exchange_rate1 = rates1[target_code]
        exchange_rate2 = rates2[target_code]

        base_name1 = currencies.get(base_code, base_code)
        base_name2 = currencies.get(base2_code, base2_code)
        target_name = currencies.get(target_code, target_code)

        result_text = (
            f"Курсы обмена на 1 {target_name}:\n\n"
            f"{base_name1}: {exchange_rate1:.4f}\n"
            f"{base_name2}: {exchange_rate2:.4f}"
        )

        mb.showinfo("Курсы валют", result_text)

    except Exception as e:
        mb.showerror("Ошибка', error 400", str(e))

currencies = {
    "USD": "Доллар США",
    "EUR": "Евро",
    "CNY": "Юань",
    "RUB": "Российский рубль",
}

pop_curr = ['EUR', 'USD', 'RUB', 'CNY']

root = Tk()
root.title("Курс валют ")
root.geometry("300x450")

# Первая базовая валюта
Label(text="Первая базовая валюта").pack(pady=10, padx=10)
base_combobox = ttk.Combobox(values=list(currencies.keys()))
base_combobox.pack()


bs_label = ttk.Label()
bs_label.pack(pady=5, padx=10)

# Вторая базовая валюта
Label(text="Вторая базовая валюта").pack(pady=10, padx=10)
base2_combobox = ttk.Combobox(values=list(currencies.keys()))
base2_combobox.pack()

bs2_label = ttk.Label()
bs2_label.pack(pady=5, padx=10)

Label(text='Целевая валюта').pack(pady=10, padx=10)
target_combobox = ttk.Combobox(values=list(currencies))
target_combobox.pack()

currency_label = ttk.Label()
currency_label.pack(pady=10, padx=10)

button = Button(text='Получить курс', command=exchange)
button.pack()

base_combobox.bind("<<ComboboxSelected>>", update_bs_label)
base2_combobox.bind("<<ComboboxSelected>>", update_bs2_label)
target_combobox.bind("<<ComboboxSelected>>", update_currency_label)

root.mainloop()
