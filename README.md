Данный код собирает информацию о системе и сохраняет ее в json файл.
Я использовал стандартные библиотеки py: platform, json и os.
Также я решил использовать библиотеку psutil т.к. это самый удобный и кроссплатформенный способ для получения информации об оперативной памяти.
Программа получает информацию о ОС: название, версия, релиз, архитектура и локальное имя; о процессоре: тип архитекуры, колличество ядер, общий процент загрузки процессора, а также загрузка каждого ядра; о памяти: общий объём ОЗУ.

import json
import os
import platform

import psutil

cpu_usage_percpu = psutil.cpu_percent(interval=1, percpu=True) ## Создает список с загруженностью каждого ядра за 1
                                                              ## сек 
for i in range(len(cpu_usage_percpu)):                      ## Перебирает элементы списка по индексу, делает запись
  cpu_usage_percpu[i] = f"cpu{i} - {cpu_usage_percpu[i]}"  ## более читаемой и понятной

syst = platform.system()                 ## Получает название ОС, но т.к. macOS в системе называется Darwin, я решил                                           ## вручную это поправить, чтобы было понятнее для тех, кто это не знает
if syst == "Darwin": os_name = "macOS"
                    
data = {      ## Создаем словарь
    "os": {
        "name": os_name,
        "version": platform.version(),
        "release": platform.release(),
        "architecture": platform.machine(),
        "hostname": platform.node()
    },
    "processor":{
        "name": platform.processor(),
        "logical_cores": os.cpu_count(),
        "cpu_usage": psutil.cpu_percent(interval=1), 
        "cpu_usage_percpu": cpu_usage_percpu
    },
    "memory": {
        "total_gb": round(psutil.virtual_memory().total / (2**30),2) ## psutil.virtual_memory делится на 2**30 с целью
    }                                                                ## перевести из бит в гб, а round() используется,
}                                                                    ## чтобы было более читаемо потому что на выходе
                                                                     ## может быть 15,999999
with open("system_info.json", "w", encoding="utf-8") as file: 
    json.dump(data,file,ensure_ascii=False, indent=4)
## Питон либо создает, либо перезаписывает файл с именем "system_info.json", "w" - означает открыть файл для записи 
## encoding="utf-8" - означает в какой кодировке будет запись, as file в - создание переменной file и положить в нее
## открытый файл, json.dump() - записывает объект в json, json.dump(что, куда, ensure_ascii=False - чтобы русские  
## символы записывались корректно, indent = 4 делает JSON красиво отформатированным, каждый уровень отступает на 4 
## пробела.)
