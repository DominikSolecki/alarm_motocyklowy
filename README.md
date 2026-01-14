# alarm_motocyklowy
**Alarm motocyklowy z powiadomieniami SMS i Bluetooth**
![_DSC3636](https://github.com/user-attachments/assets/70fcd812-42fb-4301-93f1-5969bf0ac18c)
Głównym założeniem projektu było stworzyć alarm motocyklowy z czujnikiem na bazie akcelerometru, powiadomieniami sms w razie alarmu.  
bluetooth + aplikacja na telefon dająca możliwość zmiany ustawień alarmu, kontrola stanu akumulatora w motocyklu, kodowany pilot z rolling code.    
  
**PILOT**  
Pilot zbudowany jest na ATTINY85, do nadawania sygnału wykorzystałem SYN115. Pilot zasilany jest z baterii CR2032, ATTINY85 jest cały czas uspany a nadajnik SYN115 wyłączony.  
Mikrostyki poprzez przerwanie wybudzają ATTINY85, a ten w zależności który mikrostyk go wybudził wysyła poprzez SYN115 7 ramek kodu pilota.  
Kodowanie to KEELOQ + rolling code.  
![_DSC3637](https://github.com/user-attachments/assets/364f7548-a5a3-49b8-91e0-c1b896e62867)
![_DSC3639](https://github.com/user-attachments/assets/2c6f7956-4939-43a4-9ccb-0ce00c5ffdbd)

Pobór prądu pilota w uśpieniu to 1uA  

![1768388185922](https://github.com/user-attachments/assets/7c7d8f46-946c-4cbf-9df9-d3a6dc829999)


<img width="1177" height="427" alt="obraz" src="https://github.com/user-attachments/assets/cef607a7-1d74-4af1-b122-b6059c9625cd" />


**ALARM**  
Sercem alarmu jest STM32F401RET6.STM32 pracuje w cuklach, 500ms śpi, 50ms czeka na preambułę sygnału z pilota, gdy ją odbierze przedłuża sobie czas na odebranie całej ramki.
Pilot nadaje 7 ramek, więc jeżeli zacznie nadawań w momencie gdy STM32 śpi, to któraś z ramek i tak zostanie odebrana po wybudzeniu.
Na czas uśpienia układ odbiorczy SYN480, akcelerometr LIS2dw12 są wyłączane za pośrednictwem power swicha. Pamięć eeprom, moduł sim, są włączane tylko po włączeniu alarmu.
Dzięki temu średni pobór prądu przez alarm jest na poziomie około 900uA.

![1768389966237](https://github.com/user-attachments/assets/1810ef29-84f2-4965-88ba-1c867b731bb9)
![1768389966262](https://github.com/user-attachments/assets/1709581a-1580-405c-915d-cb88fed4efb3)

Alarm "pilnuje" pozycji motocykla oraz siedziska. Jako czujnik ruchu wykorzystałem akceloremtr LIS2DW12, dzięki czemu alarm wykrywa ruch we wszystkich osiach.  
Jako czujnik otwarcia siedziska może być wykorzystany kontaktron + magnes, lub zwykła krańcówka zwierająca do masy motocykla.   
Po wykryciu ruchu, lub otwarcia siedziska włącza się alarm, steruje on kierunkowskazami i klaksonem generując sygnał o częstotliwości 1Hz, oraz włącza +12V dla syreny oraz przekaźnika do odcięcia zapłonu.  
Zostaje wysłany sms z powiadomienie o alarmie oraz co go wyzwoliło, zdarzenie to jest zapisywanie w historii wraz z datą i godziną pobraną z modułu sim. W alarmie jako moduł sim wykorzystałem moduł LTE A7682E.  
Uzbrojenie alarmu jest sygnalizowane przez krótki sygnał dla kierunkowskazów oraz klaksonu, rozbrojenie przez 2 krótkie sygnały dla kierunkowskazów i klaksonu.  
Jeżeli Alarm jest już uzbrojony lub rozbrojony schemat jet taki sam z tym że migają tylko kierunkowskazy.  
W celu ochrony akumulatora w motocyklu są dwa progi powiadomień. Domyślnie to 12V dla powiadomienia o potrzebie podładowania akumulatora i 11V dla powiadomienia o rozładowanym akumulatorze.  
Częstotliwość pomiaru napięcia akumulatora można ustawić w aplikacji. Dzielnik napięcia jest włączony poprzez mosfet tylko na czas pomiaru napięcia gdy nadejdzie na to pora.   
  
![_DSC3633](https://github.com/user-attachments/assets/849eb1b5-ef54-403f-a70e-0ba3342f2fb7)
![_DSC3631](https://github.com/user-attachments/assets/c177064f-b2ab-417a-932d-d361ce2300fb)
  
  
<img width="1403" height="617" alt="obraz" src="https://github.com/user-attachments/assets/752c497f-bef9-4805-986d-9468dd6500a3" />
  
    
**APLIKACJA**  
Moduł Bluetooth jest włączany tylko po przekręceniu stacyjki. Ponowne wyłączenie stacyjki, lub uruchomienie silnika wyłącza moduł bluetooth.  
Moduł bluetooth to AT-09, czyli w technologi BLE. Gdy bluetooth jest włączony można się połączyć aplikacją w telefonie z alarmem.  
Komunikacja pomiędzy alarmem a aplikacją jest kodowana. Gdy łączymy się po raz pierwszy, czy to po instalacji aplikacji, lub wyczyszczeniu jej pamięci trzeba podać i sześciu cyfrowy numer z obudowy alarmu.  
Zapobiega to dostępu z innego telefonu z zainstalowaną aplikacją. Po sparowaniu aplikacji z alarmem należy podać numer pin.  
Przy pierwszym uruchomieniu domyślny, można go zmienić w każdej chwili w aplikacji. Trzykrotne wpisanie błędnego pinu spowoduje wysłanie sms z przypomnieniem numeru pin.  
Aplikacja daje dostęp do ustawień alarmu oraz testów diagnostycznych. Można w niej ustawić:  
*numer telefonu dla powiadomień sms  
*włączyć / wyłączyć numer USSD do sprawdzenia salda karty jeżeli korzystamy z karty prepaid  
*progu napięcia powiadomienia potrzeby podładowania akumulatora  
*progu napięcia powiadomienia o rozładowanym akumulatorze  
*częstotliwości kontroli napięcia akumulatora  
*czułość akcelerometru  
*czas trwania impulsu powiadomienia dla kierunkowskazów i klaksonu  
*czas trwania alarmu  
*numer PIN  
*zaprogramować pilot  
*uzbroić / rozbroić alarm  
  
W diagnostyce możemy przetestować:  
*sterowanie kierunkowskazami  
*sterowanie klaksonem  
*sterowanie syreną  
*sprawdzić działanie akcelerometru w raz z wykrywaniem ruchu  
*sprawdzić działanie modułu sim wraz z wysłaniem testowego sms o raz sprawdzić saldo karty sim  
*odczytać historie dziesięciu ostatnich zdążeń takich jak wykrycie ruchu, otwarcie siedziska, powiadomienia do stanie akumulatora, przypomnienie numeru pin.    


![1768393070829](https://github.com/user-attachments/assets/1dea510e-07ee-40c3-8ceb-e9e228757397)
![1768393070820](https://github.com/user-attachments/assets/fc34bcf7-7d8e-4bfa-baf0-151ad5b841d9)
![1768393070806](https://github.com/user-attachments/assets/5702f8ae-d961-4c9b-ab27-3a08f098a04c)
![1768393070794](https://github.com/user-attachments/assets/5da2672f-2290-4e22-bbb5-4369f3dc2cbd)
![1768393070773](https://github.com/user-attachments/assets/12f528ea-53a6-4c02-bee6-5adbca04869b)
![1767012429120](https://github.com/user-attachments/assets/40322275-8629-43bd-a048-085175d0d01d)
