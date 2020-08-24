
# Обновление с HackRF One до bladeRF x40

Из Нуанда :

* [bladeRF x40](https://www.nuand.com/blog/product/bladerf-x40/) , $ 420 (не включая [недавние скидки](https://www.reddit.com/r/RTLSDR/comments/4p3l15/124mhz_of_bandwidth_with_bladerf_x40_for_199/) )

* [Дело BladeRF](https://www.nuand.com/blog/product/bladerf-case/) , $ 20

* [2x 2,4 ГГц Всенаправленные SMA-антенны](https://www.nuand.com/blog/product/2-4ghz-antennas/) , $ 20

Давайте сравним с конкурентом от [Great Scott Gadgets](https://greatscottgadgets.com) :

* [HackRF One](https://greatscottgadgets.com/hackrf/) , $ 300

* [ANT500 Телескопическая антенна 75 МГц — 1 ГГц](https://greatscottgadgets.com/ant500/) , $ 30

BladeRF действительно ли лучше чем HackRF One? Или разница только в цене?

## Сборка и аксессуары

BladeRF приходит без чехла, но [прозрачный футляр](https://www.nuand.com/blog/product/bladerf-case/) можно докупить отдельно, или вы можете сделать его сами. HackRF, напротив, имеет литой пластиковый корпус:

![bladeRF в корпусе bladeRF (слева), по сравнению с HackRF во встроенном корпусе (справа)](https://cdn-images-1.medium.com/max/2000/1*i3INAMem5CgF2dcR0SRQUg.png)*bladeRF в корпусе bladeRF (слева), по сравнению с HackRF во встроенном корпусе (справа)*

Ни в одном из них в комплекте не имеется антенны, хотя bladeRF поставляется с двумя кабелями SMA. Я еще не использовал их, вместо этого предпочел подключить антенны. HackRF рекомендует ANT500, который можно регулировать по длине для оптимизации на 75 МГц — 1 ГГц, и требуется только одна антенна, поскольку, будучи наполовину он кажется своеобразным, HackRF передает и принимает через один и тот же порт антенны, но а Nuand продает пару антенн 2,4 ГГц, по одной для портов приема и передачи.

Оба используют популярный порт [разъема SMA](https://en.wikipedia.org/wiki/SMA_connector) . В этом обзоре я использую антенны ANT500 и 2,4 ГГц.

bladeRF пришёл с кабелем USB 3.0 SuperSpeed ​​с позолоченными разъемами от USB-A до USB-B. HackRF с микро-USB 2 кабелем. У меня есть только USB 2 на моем компьютере, но я с нетерпением жду возможности использовать более высокую скорость USB 3 (5 Гбит / с SuperSpeed [>](https://en.wikipedia.org/wiki/USB#Signaling) 480 Мбит / с для USB 2.0 High-Speed) для большей частоты дискретизации или пропускной способности в возможно отдаленном будущем, когда я усовершенствую свои компьютерные характеристики. Многие компьютеры уже по стандарту идут с USB 3, но всё же поэтому приятно то, что bladeRF может использовать его через USB 2.

(Интересно, что Ettus разработала USRP, взаимодействующие с хостовым компьютером с использованием [Gigabit Ethernet](https://www.ettus.com/product/category/USRP-Networked-Series) (USRP N210 / N200), чтобы преодолеть ограничения скорости USB 2, хотя теперь у них есть [USRP B210 / B200](https://www.ettus.com/product/category/USRP-Bus-Series) с использованием 3.0)

BladeRF поддерживает несколько других аксессуаров, но я не купил ни одного из них, хотя мог, они продавали: [усилитель](https://www.nuand.com/blog/product/amplifier/) , [плату расширения GPIO](https://www.nuand.com/blog/product/xb-gpio-gpio-expansion-board/) (требуется [другой корпус](https://www.nuand.com/blog/product/bladerf-and-expansion-board-case/) , если вам нужен чехол), [блок питания](https://www.nuand.com/blog/product/power-supply/) и [преобразователь LF / MF / HF / VHF](https://www.nuand.com/blog/product/hf-vhf-transverter/) . Блок питания может оказаться полезным для автономной работы, но вовсе не необходимым, потому что bladeRF может питаться через USB порт.

## Начальная настройка

Вставил bladeRF x40 в корпус bladeRF, ввинтил четыре винта Phillips, подключил антенны 2,4 ГГц, подключил USB-кабель к bladeRF и моему компьютеру. Знаю, что до [gqrx](http://gqrx.dk) я использовал с HackRF, а также вышло это :

*UHD Error:
bladerf_get_timestamp() returned -1 — An unexpected failure occurred
FATAL: getHardwareTime() -1 — An unexpected failure occurred*

“D1” загорается на плате, но светодиоды LED1–3 так и остались тёмными.

Обновлён до последней прошивки [fx3](https://www.nuand.com/fx3.php) (v1.9.0 at this time):

*bladeRF-cli -f bladeRF_fw_latest.img*

Но не хватало только загрузки [fpga](https://www.nuand.com/fpga.php) (используется v0.5.0):

*bladeRF-cli — load-fpga hostedx40-latest.rbf
Loading fpga…
[INFO @ version_compat.c:85] Firmware version (v1.9.0) is newer than entries in libbladeRF’s compatibility table. Please update libbladeRF if problems arise.
[INFO @ version_compat.c:116] FPGA version (v0.5.0) is newer than entries in libbladeRF’s compatibility table. Please update libbladeRF if problems arise.
Done.*

После выполнения этого шага загорается светодиод 1–3, после чего можно запустить gqrx. LED2 постоянно мигает, когда я запускаю gqrx, указывая на то, что устройство bladeRF открыто.

### Автозагрузка FPGA

*bladeRF-cli — load-fpga* загружает только битовый поток FPGA, но он не будет сохраняться после перезапуска bladeRF. С одной стороны это полезно, поскольку могут быть загружены [другие образы ПЛИС](https://www.nuand.com/fpga.php) , такие как аппаратный декодер ADS-B (adsbx40.rbf), который нужен для более высокой производительности, чем решения, основанные на [большем](https://medium.com/@rxseger/flight-tracking-with-ads-b-using-dump1090-mutability-and-hackrf-one-on-os-x-3e2d5e192d6f) программном обеспечении, например, [dump1090-mutable с RTL-SDR](https://medium.com/@rxseger/flight-tracking-with-ads-b-using-dump1090-mutability-and-hackrf-one-on-os-x-3e2d5e192d6f) .

Но для общего использования образ hostedx40-latest.rbf можно записать во флэш-память для автозагрузки при загрузке: *bladeRF-cli — flash-fpga hostedx40-latest.rbf*

Перезагрузка bladeRF путем удаления и повторного подключения USB-кабеля подтверждает, что FPGA загружена успешно, светятся светодиоды и gqrx работает без ошибок.

На данный момент разница между двумя моделями bladeRF, [x40](https://www.nuand.com/blog/product/bladerf-x40/) (420 долларов) и [x115](https://www.nuand.com/blog/product/bladerf-x115/) составляет (650 долларов): размер FPGA Cyclone 4 в kLE. Nuand говорит, что более крупная FPGA «обеспечивает дополнительное пространство для аппаратных ускорителей и цепочек обработки сигналов, включая FFT, турбодекодеры, модуляторы / фильтры передачи и корреляторы приема данных для пакетных модемов», но до сих пор с начала работы с FPGA 40 kLE у меня всё хорошо.

## Калибровка

Потом я [откалибровал HackRF](https://medium.com/@rxseger/sdr-calibration-via-gsm-fcch-using-kalibrate-and-lte-cell-scanner-on-rtl-sdr-and-hackrf-193a7fb8a3eb) , управление gqrx ввода> Freq. Корректировку поставил ​​на -7,7 промилле. Это не точно для bladeRF, поэтому я вернул его обратно к 0.0 ppm. В gqrx его придется изменить обратно вручную, если я переключусь между устройствами (TODO: возможно автоматизировать это?

bladeRF откалиброван на заводе до 20ppb, но для подтверждения, давайте попробуем [kalibrate-bladeRF](https://github.com/Nuand/kalibrate-bladeRF) . Это еще [один](https://github.com/scateu/kalibrate-hackrf/issues/9) форк почтенной калибровки, среди [kalibrate-hackrf](https://github.com/scateu/kalibrate-hackrf) и kalibrate-rtl, но для bladeRF.

Но сначала, *bladeRF-cli -i* можно использовать для просмотра заводской калибровки:

*Калибровка VCTCXO DAC: 0x9040*

Попытка скомпилировать kalibrate-bladeRF на OS X завершилась неудачно, открыла запрос [на](https://github.com/Nuand/kalibrate-bladeRF/pull/8) удаление, исправляющий ошибку компиляции [назначения комплексного номера](https://github.com/Nuand/kalibrate-bladeRF/pull/8) , перенесенную из kalibrate-hackrf. Из [руководства по использованию kalibrate-bladeRF](https://www.youtube.com/watch?v=ym81vhSoo44) :

*./src/kal -s GSM850 -m2*

Использование антенн Nuand 2,4 ГГц: каналы не найдены.

![](https://cdn-images-1.medium.com/max/2000/1*1y0wJg3WqIoO1_GIOeH4vQ.png)

Попробуйте с антенной ANT500 для RX на bladeRF, вроде разницы нет.

Я не понял почему у меня не получилось так как я хотел, хотя использовал Калибрат для поиска и калибровки по каналам GSM. К счастью, для калибровки по частотам LTE есть инструмент LTE-Cell-Scanner, который я могу легко получить по моему местоположению, и @JiaoXianjun, который расширил его для поддержки bladeRF. @cgommel добавил вывод PPM, я слил изменения и другие изменения в свою ветку на [rxseger / LTE-Cell-Scanner](https://github.com/rxseger/LTE-Cell-Scanner) .

Аппаратная поддержка LTE-Cell-Scanner настраивается во время сборки, чтобы включить поддержку bladeRF и HackRF:

*mkdir build
cd build
cmake ../ -DUSE_BLADERF=1 -DUSE_HACKRF=1
make*

(TODO: собственная формула, чтобы включить bladeRF и HackRF и использовать rxseger / LTE-Cell-Scanner?)

Затем сканирование для LTE ячеек:

*./src/CellSearch — freq-start 715e6 — freq-end 768e6*

результаты в:

*DPX CID A fc freq-offset RXPWR C nRB P PR CrystalCorrection ppm
FDD 459 2 739M -458h -12.1 N 50 N one 0.999999380493036 -0.62
FDD 237 2 751M -475h -12.1 N 50 N one 0.999999367581029 -0.632*

В пределах менее чем 1 ч / млн, — нормально. Некоторое программное обеспечение SDR даже не успевает указывать на ошибки среди миллиона, поэтому я буду считать это устройство хорошо откалиброванным прямиком из коробки. Никаких дополнительных настроек не требуется.

## Диапазон частот

BladeRF от 300 МГц до 3,8 ГГц — это меньший диапазон, чем HackRF от 0,1 МГц до 6 ГГц. Он охватывает весь [диапазон УВЧ](https://en.wikipedia.org/wiki/Ultra_high_frequency) , но не УКВ-диапазон или ниже, включая ЧМ или АМ (средневолновые) станции, [телеканалы 2–13](https://en.wikipedia.org/wiki/Television_channel_frequencies#North_and_South_America_.28most_countries.29.2C_South_Korea.2C_Taiwan_and_the_Philippines) , некоторые (но не все) двусторонние радиостанции общественной безопасности и прочие.

В моем ~ / .config / gqrx / bookmarks.csv у меня было 113 закладок на частотах ниже 300 МГц, от работы с HackRF и маркировки различных интересных сигналов, и только выше 50.

Нашёл компромисс — лучшее качество в меньшем диапазоне. Трансвертер можно использовать для передачи на более низкие частоты, но я не тестировал с ним. [XB-200 LF/MF/HF/VHF.](https://www.nuand.com/blog/product/hf-vhf-transverter/)

[Трансвертер](https://www.nuand.com/blog/product/hf-vhf-transverter/) «s расширенный диапазон 60 кГц — 300 МГц вполне привлекательно, но это было в наличии на момент написания этой статьи. Трансвертер также [не совместим](http://forums.nuand.com/forums/viewtopic.php?f=6&t=3658) с обычным корпусом, без модификации. Поэтому я все еще наслаждаюсь удобством широкого спектра HackRF для съемки широкого спектра.

### Ниже UHF с другими устройствами

Для сравнения рассмотрим HackRF. В презентации [SDR Tricks с hackrf Майклом Оссманном (Michael Ossmann) в Defcon Wireless village 2014](https://www.youtube.com/watch?v=4Lgdtr7ylNY&t=13m44s)цель проекта была от 100 МГц до 6 ГГц, но после компонентов нижний предел был расширен до 30 МГц, а после выхода бета-версии — до 10 МГц. Майкл Оссманн говорит, что HackRF может принимать сигналы даже на более низких частотах, но при меньшей мощности даже RFID-петля 125 кГц может считываться с близкого расстояния:

![Питание на низких частотах с HackRF от [SDR Tricks](https://www.youtube.com/watch?v=4Lgdtr7ylNY&t=13m44s)](https://cdn-images-1.medium.com/max/2000/1*aAcom463fGDYTpgVfWu2lg.png)*Питание на низких частотах с HackRF от [SDR Tricks](https://www.youtube.com/watch?v=4Lgdtr7ylNY&t=13m44s)*

[Марио на Hackaday, 25 марта 2016 года](http://hackaday.com/2013/08/01/hackrf-or-playing-from-30-mhz-to-6-ghz/#comment-2965364) сообщает:

*«Я недавно попробовал HackRF One в диапазоне AM (530–1710 кГц), используя 43-футовую вертикальную антенну, и очень доволен приемом. Не рискнул ниже этого, например, 285–325 кГц, где находятся маяки DGPS. Хорошая антенна — залог хорошего приема ».*

Радиодиапазоны Википедии [по частоте](https://en.wikipedia.org/wiki/Radio_spectrum#By_frequency) показывают, что ниже, еще дальше:

* 300–30 МГц ( VHF): FM, ТВ, УВД, радиолюбитель, наземное / морское, метеорологическое радио

* 30–3 МГц ( HF): коротковолновая, CB, RFID, небесная волна, морская, радиотелефонная связь

* 3000–300 кГц (MF): радиопередачи AM, любительское радио

* 300–30 кГц ( LF): навигация, время на часах, RFID, любительское радио, DGPS

* 30–3 кГц (VLF): навигация, время, подводная лодка, геофизика

* 3000–300 Гц (ULF): подводная связь, мины

* 300–30 Гц (SLF): подводные лодки

* 30–3 Гц (ELF): подводные лодки

Преобразователь bladeRF XB-200 будет охватывать VHF / HF / MF / LF.

Фундаментальная минимальная частота или, что эквивалентно, максимальная длина волны, ограничена только размером вселенной — около 0,7 аттогерца для [ширины наблюдаемой вселенной](http://www.wolframalpha.com/input/?i=c%2F%2846e9+lightyears%29+to+zeptohertz) — но практический минимум ограничен требуемым размером антенны, примерно четверть длины волны. Большие антенны могут быть использованы для подводных лодок, но не в вашем кармане или на вашем столе. Тем не менее, диапазон килогерц является домом для многих интересных радиолюбителей.

Исследование [Ettus](https://www.ettus.com/product) USRP поддерживает DC — 6 ГГц, но стоит тысячи долларов. Другие повышающие преобразователи для других устройств, кроме XB-200:

* [AirSpy Spyverter](https://airspy.com/spyverter/) (60 долл. США): от 1 кГц до 60 МГц

* [NooElec Ham It Up](http://www.amazon.com/NooElec-Ham-Up-v1-3-Upconverter/dp/B009LQT3G6) ($ 50): «до 100 кГц и ниже»

[SDRplay RSP](http://sdrplay.com) ($ 149) непосредственно поддерживает до 100 кГц. Младшая RTL-SDR может быть улучшена с помощью [прямой модификации выборки](http://www.rtl-sdr.com/rtl-sdr-direct-sampling-mode/) с различными результатами.

Низкочастотные SDR могут быть интересной областью будущих исследований, но их придется подождать в другой раз. На данный момент bladeRF x40 без плат расширения подходит для сверхвысоких частот.

## Прием цифрового телевидения ATSC с bladeRF

Последующая деятельность по итогам [приема цифрового телевидения ATSC с SDR](https://medium.com/@rxseger/receiving-atsc-digital-television-with-an-sdr-76b03a863fea) , где я использовал HackRF One. bladeRF может принимать UHF-каналы (> 14), но не VHF без трансвертера, но в моей области есть более интересные UHF-каналы, чем VHF-каналы, поэтому это не является серьезным недостатком.

Перезапустите тот же файл file_atsc_rx2.py .. ничего — точнее, без вывода видео. Журнал:

*UHD Warning: Setting DC offset compensation is not possible on this device.
UHD Warning: Setting DC offset is not possible on this device.
UHD Warning: Setting IQ balance is not possible on this device.
[INFO @ bladerf.c:648] Clamping bandwidth to 1500000Hz*

1,5 МГц недостаточно для приема ATSC (6 МГц).

Похоже, из [этого кода](https://github.com/Nuand/bladeRF/blob/d4642214cdda50edcd005619e53f069f1089ac92/host/libraries/libbladeRF/src/bladerf.c#L732) :

*} else if (bandwidth > BLADERF_BANDWIDTH_MAX) {
 bandwidth = BLADERF_BANDWIDTH_MAX;
 log_info(“Clamping bandwidth to %dHz\n”, bandwidth);*

НО почему значение BLADERF_BANDWIDTH_MAX должно составлять 1,5 МГц, если [полоса пропускания до 124 МГц](https://www.nuand.com/blog/large-bandwidth/) достижима (с помощью быстрой настройки)? Фактический _MAX — 28 МГц, *минимальный* — 1,5 МГц. [Канал 36](https://en.wikipedia.org/wiki/Television_channel_frequencies#Americas_.28most_countries.29.2C_South_Korea.2C_Taiwan_and_the_Philippines) имеет частоту 602 МГц — 608 МГц с центром на частоте 605 МГц, но без достаточной полосы пропускания границы не видны:

![Усеченный сигнал ATSC](https://cdn-images-1.medium.com/max/2000/1*NJYHQFgxXElPTEtbhIUZOQ.png)*Усеченный сигнал ATSC*

Оказывается, переменная ‘Ch0: Bandwidth (Hz)’ в источнике osmocom не была установлена, 0 по умолчанию равен 1.5e6. Изменил его на 8e6 (8 МГц), и полный сигнал виден, включая пилот-сигнал 602,309441 МГц:

![Канал ATSC 36 (602–608 МГц), полученный с bladeRF](https://cdn-images-1.medium.com/max/2000/1*aqLxHHjP7lSERFzU5JgozQ.png)*Канал ATSC 36 (602–608 МГц), полученный с bladeRF*

На этом этапе поток MPEG может быть декодирован, но со значительными ошибками:

![Неудачная попытка воспроизведения ATSC с воздуха через антенну ANT500 и bladeRF](https://cdn-images-1.medium.com/max/2000/1*s6KgLLPpYeqxSsN1_OLXdA.png)*Неудачная попытка воспроизведения ATSC с воздуха через антенну ANT500 и bladeRF*

С помощью HackRF я нашел разумные результаты, когда источник osmocom настроен на усиление RF 14, усиление IF 16 и усиление BB 20. [bladeRF получает](https://github.com/Nuand/bladeRF/wiki/Getting-Started:-Verifying-Basic-Device-Operation#Gain) контроль над lnagain, rxvga1, rxvga2. Экспериментируя со значениями, усиление RF 30, IF 20 и BB 20 дает видимую картину:

![Полученный сигнал ATSC с bladeRF, отображаемый с помощью VLC](https://cdn-images-1.medium.com/max/2000/1*xgE8ba_hF-YQ97BH8zBDww.png)*Полученный сигнал ATSC с bladeRF, отображаемый с помощью VLC*

В этот момент я столкнулся с той же проблемой, описанной в [Прием цифрового телевидения ATSC с SDR](https://medium.com/@rxseger/receiving-atsc-digital-television-with-an-sdr-76b03a863fea) : прерывистое видео, demux и atsc_fs_checker PN63. Поскольку это происходит как с HackRF, так и с bladeRF, вероятно, это проблема не самого устройства, а моей стартовой антенны ANT500. Еще не дошли до этого, но использование усиленной антенны, предназначенной для HDTV, должно дать лучшие результаты.

## Последние мысли

HackRF One остается моим фаворитом для исследования случайных сигналов благодаря широкому диапазону частот 0,1–6 ГГц, а семейство RTL-SDR не может сравниться с недорогим введением в SDR. Тем не менее:

bladeRF предлагает более узкий частотный диапазон 300 МГц — 3,8 ГГц без платы расширения преобразователя, но с 12-битным АЦП вместо 8-битной собственной полосы пропускания 28 МГц вместо 20 МГц или 2–4 МГц, вместо дуплекса полудуплекс или нет передачи, USB 3 вместо USB 2, 40 kLE FPGA, дополнительное автономное питание и расширение GPIO.

Этот пост в блоге в качестве первого взгляда на bladeRF на самом деле не вошел в его преимущества, но я с нетерпением жду возможности использовать их в будущем с другими проектами, ссылками: [Projects, Papers, and Blogs](https://github.com/Nuand/bladeRF/wiki#Projects_Papers_and_Blogs) , [OpenBTS](https://github.com/Nuand/bladeRF/wiki/Minimalistic-build-and-run-test-for-OpenBTS-5) .

**Обновление 2016/07/07** : Используя усиленную HDTV-антенну, подключенную к SMA-порту bladeRF с помощью [коаксиального коаксиального кабеля DHT Electronics](https://www.amazon.com/gp/product/B00CQ35NOW) в сборе, переходник SMA “папа” [к F “](https://www.amazon.com/gp/product/B00CQ35NOW) мама” теперь может получать довольно плавное видео, иногда!

![Плавное видео, снятое по воздуху из ATSC в файл на RAM-диске](https://cdn-images-1.medium.com/max/2000/1*L-ygvebrmvwIxP_GDG_Esg.png)*Плавное видео, снятое по воздуху из ATSC в файл на RAM-диске*

Но он очень чувствителен к использованию процессора, так как переключение на другое приложение вызывает переполнение (000 ..) в приемнике, так как декодер не может идти в ногу. Этот программный ATSC-приемник не является практической заменой для просмотра телевизора по сравнению с аппаратным декодером в телевизоре (или [USB-ATSC TV-флешке](http://www.newegg.com/Product/Product.aspx?Item=9SIA0AJ3MD4997) , также на аппаратной основе), но важно знать, что это возможно , VHF радиолюбитель и другие сигналы также могут быть четко приняты через эту антенну HDTV, подключенную к bladeRF, поэтому я, вероятно, сохраню эту настройку в течение некоторого времени.
