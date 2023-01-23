# Projekt bazy danych obiektu sportowego - parku wodnego

## Założenia projektu
Przygotowana baza danych może służyć jako szablon, pomoc w zarządzaniu obiektem sportowym o charakterze wodnym.   W bazie danych możemy gromadzić informacje dotyczące klientów obiektu, biletów kart sportowych, szatni, ale również możemy ustalić grafik rezerwacji basenów, nauki pływania. Dzięki odpowiedniemu pogrupowaniu danych, możemy generować raporty dzienne/miesięczne oraz proste statystyki.
Nie jest to jednak profesjonalna baza danych, nie uwzględniamy w niej m.in. kwestii finansowych.

## Pielęgnacja bazy danych
Potencjalna utrata zgromadzonych danych, może przysporzyć wielu problemów. Obiekty sportowe często obligowane są do raportowania swojego działania do inspekcji sanitarnych. Także aby starać się o różnego rodzaje dotacje, konieczne jest wykazanie m.in. rentowności i historii obiektu, dlatego nie możemy sobie pozwolić na utratę danych. Aby uniknąć takiej sytuacji zalecane jest wykonywanie kopii zapasowej raz na tydzień, najlepiej w godzinach zamknięcia.

## Schemat bazy danych
![enter image description here](https://i.imgur.com/Eznf9xR.png)

## Widoki
Przygotowane widoki służą za prezentowanie często wybieranych danych.

``` SQL
--1 Wyświetla wszystkie dane wszystkich aktualnych rezerwacji wraz z nazwami rezerwowanych basenów.
CREATE VIEW RezerwacjeBasen AS
	SELECT R.*, B.[Nazwa Basenu] FROM Rezerwacje R 
	JOIN Baseny B ON(B.ID_Basenu = R.ID_Basenu)


--2 Wyświetla wszystkich pracowników, którzy zarabiają powyżej średniej.
CREATE VIEW PensjePowSredniej AS
	SELECT * FROM Pracownicy
	WHERE Wynagrodzenie > ( SELECT AVG(Wynagrodzenie) FROM Pracownicy )


--3 Wyświetla wszystkich aktualnych klientów wraz z nazwą biletu i kwotą jaką zapłacili za pobyt.
CREATE VIEW Klienci_ile_zaplacili AS
	SELECT K.ID_Klienta, B.NazwaBiletu, B.[Czas w h], ROUND(B.Cena * CAST((1-B.Znizka) AS SMALLMONEY),2) AS [Kwota do zapłaty]
	FROM [Aktualni klienci] K 
	JOIN Bilety B ON(K.ID_Biletu = B.ID_Biletu)


--4 Wyświetla wszystkie wolne szafki.
CREATE VIEW Wolne_szafki AS
	SELECT S.* FROM Szafki S 
	WHERE S.Status = 'wolna'


--5 Wyświetla wszystkie zjeżdżalnie wraz z parametrami i nazwą basenu, do którego wpadają.
CREATE VIEW Gdzie_wpada_zjeżdżalnia AS
	SELECT Z.ID_Zjeżdżalni, Z.Kolor, Z.Długość, Z.[Limit wiekowy], B.[Nazwa Basenu] FROM Zjeżdżalnie Z
	JOIN Baseny B ON(Z.ID_Basenu = B.ID_Basenu)


--6 Wyświetla liczbę klientów, która każdego dnia skorzystała z obiektu.
CREATE VIEW Raport_dzienny AS
	SELECT CAST([Godzina wejścia] AS DATE) AS [Data], COUNT(*) AS [Liczba klientów] FROM [Historia wejść]
	GROUP BY CAST([Godzina wejścia] AS DATE)


--7 Wyświetla wszystkie aktywności wraz z danymi a także imieniem i nazwiskiem prowadzącego daną aktywność.
CREATE VIEW Prowadzący_Aktywności AS
	SELECT A.ID_Aktywności, A.[Nazwa aktywności], P.[Imię i nazwisko] FROM Aktywności A 
	JOIN Pracownicy P ON(P.ID_Pracownika = A.ID_Pracownika)


--8 Wyświetla dodatkowe informacje o klientach, tj. aktywność jaką wykupili, nazwa karty sportowej z jakiej skorzystali.
CREATE VIEW Klienci_dod_info AS
	SELECT AK.ID_Klienta, AK.ID_Aktywności, A.[Nazwa aktywności], K.ID_Karty, K.[Nazwa karty] FROM [Aktualni klienci] AK
	LEFT JOIN Aktywności A ON(A.ID_Aktywności = AK.ID_Aktywności)
	LEFT JOIN [Karty sportowe] K ON(AK.ID_Karty = K.ID_Karty)


--9 Wyświetla ile aktualnie osób korzysta z danych aktywności.
CREATE VIEW Aktywności_aktualnie AS
	SELECT A.[Nazwa aktywności] AS [Nazwa aktywności], COUNT(*) AS [Aktualna liczba osób] FROM [Aktualni klienci] AK
	LEFT JOIN Aktywności A ON(A.ID_Aktywności = AK.ID_Aktywności)
	GROUP BY A.[Nazwa aktywności]


--10 Wyświetla ile klientów weszło na daną kartę sportową do tej pory.
CREATE VIEW Karty_sportowe_stats AS
	SELECT K.[Nazwa karty] AS [Nazwa karty], COUNT(*) AS [Liczba wejść na kartę] FROM [Historia wejść] H
	JOIN [Karty sportowe] K ON(K.ID_Karty = H.ID_Karty)
	GROUP BY K.[Nazwa karty]
```

## Skrypt tworzący bazę danych

``` SQL
IF OBJECT_ID('ParkWodny', 'U') IS NOT NULL
    DROP DATABASE ParkWodny
CREATE DATABASE ParkWodny
GO
USE ParkWodny
```
## Skrypt tworzący tabele
```SQL
CREATE TABLE Osoby(
	ID_Osoby INT NOT NULL,
	Imię VARCHAR(256) NOT NULL,
	Nazwisko VARCHAR(256) NOT NULL,
	Płeć VARCHAR(256) NOT NULL,
	[Data urodzenia] DATE NOT NULL,
	PRIMARY KEY(ID_Osoby)
)


CREATE TABLE Bilety(
	ID_Biletu INT NOT NULL,
	NazwaBiletu VARCHAR(256) NOT NULL,
	[Czas w h] FLOAT NOT NULL,
	Cena SMALLMONEY NOT NULL,
	Znizka float,
	PRIMARY KEY(ID_Biletu)
)


CREATE TABLE Pracownicy(
	ID_Pracownika INT NOT NULL,
	[Imię i nazwisko] VARCHAR(256) NOT NULL,
	Posada VARCHAR(256) NOT NULL,
	[Rok zatrudnienia] INT NOT NULL,
	Wynagrodzenie SMALLMONEY NOT NULL,
	PRIMARY KEY(ID_Pracownika)
)


CREATE TABLE Aktywności(
	ID_Aktywności INT NOT NULL,
	[Nazwa aktywności] VARCHAR(256) NOT NULL,
	ID_Pracownika INT REFERENCES Pracownicy NOT NULL,
	Cena SMALLMONEY NOT NULL,
	[Limit wiekowy] INT NOT NULL,
	PRIMARY KEY(ID_Aktywności)
)


CREATE TABLE Szafki(
	ID_Szafki INT NOT NULL,
	[Status] VARCHAR(256) NOT NULL,
	Rozmiar VARCHAR(256) NOT NULL,
	Kolor VARCHAR(256) NOT NULL,
	PRIMARY KEY(ID_Szafki)
)


CREATE TABLE Baseny(
	ID_Basenu INT NOT NULL,
	[Nazwa Basenu] VARCHAR(256) NOT NULL,
	[Temp wody] INT NOT NULL,
	[Max liczba osób] INT NOT NULL,
	Głębokość FLOAT NOT NULL,
	PRIMARY KEY(ID_Basenu)
)


CREATE TABLE Zjeżdżalnie(
	ID_Zjeżdżalni INT NOT NULL,
	ID_Basenu INT NOT NULL REFERENCES Baseny,
	Kolor VARCHAR(256) NOT NULL,
	Długość INT NOT NULL,
	[Limit wiekowy] INT NOT NULL,
	PRIMARY KEY(ID_Zjeżdżalni)
)


CREATE TABLE Rezerwacje(
	ID_Rezerwacji INT NOT NULL,
	ID_Basenu INT NOT NULL REFERENCES Baseny,
	[Godzina rezerwacji] DATETIME NOT NULL,
	PRIMARY KEY(ID_Rezerwacji)
)

CREATE TABLE [Karty sportowe](
	ID_Karty INT NOT NULL,
	[Nazwa karty] VARCHAR(256) NOT NULL,
	[Wejscia na miesiac] INT NOT NULL,
	[Cena brutto] INT NOT NULL,
	PRIMARY KEY(ID_Karty)
)


CREATE TABLE [Aktualni klienci](
	ID_Klienta INT IDENTITY(100,1),
	ID_Osoby INT NOT NULL REFERENCES Osoby,
	ID_Biletu INT NOT NULL REFERENCES Bilety,
	ID_Szafki INT REFERENCES Szafki,
	ID_Karty INT REFERENCES [Karty sportowe],
	ID_Aktywności INT REFERENCES Aktywności,
	PRIMARY KEY(ID_Klienta)
)


CREATE TABLE [Historia wejść](
	ID_Wejścia INT IDENTITY(1,1),
	ID_Klienta INT NOT NULL,
	ID_Osoby INT NOT NULL REFERENCES Osoby,
	[Godzina wejścia] DATETIME NOT NULL,
	ID_Karty INT REFERENCES [Karty sportowe],
	PRIMARY KEY(ID_Wejścia)
)
```

## Wyzwalacze


```SQL
IF OBJECT_ID('Dodaj_do_historii', 'TR') IS NOT NULL
    DROP TRIGGER Dodaj_do_historii
GO
CREATE TRIGGER Dodaj_do_historii ON [Aktualni klienci]
	AFTER INSERT
	AS
	BEGIN
		DECLARE @KlientID INT, @KlientKarta INT
		
		SELECT @KlientID = INSERTED.ID_Klienta FROM INSERTED
		SELECT @KlientKarta = INSERTED.ID_Karty FROM INSERTED

		INSERT INTO [Historia wejść] VALUES (@KlientID, GETDATE(), @KlientKarta)
	END
```

```SQL
IF OBJECT_ID('Zajmij_szafkę', 'TR') IS NOT NULL
    DROP TRIGGER Zajmij_szafkę
GO
CREATE TRIGGER Zajmij_szafkę ON [Aktualni Klienci]
	AFTER INSERT
	AS
	BEGIN
		DECLARE @SzafkaID INT
		SELECT @SzafkaID = INSERTED.ID_Szafki FROM INSERTED

		UPDATE Szafki SET [Status] = 'zajęta'
		WHERE ID_Szafki IN (@SzafkaID)
	END
```


``` SQL
IF OBJECT_ID('Zwolnij_szafkę', 'TR') IS NOT NULL
    DROP TRIGGER Zwolnij_szafkę
GO
CREATE TRIGGER Zwolnij_szafkę ON [Aktualni Klienci]
	AFTER DELETE
	AS
	BEGIN
		DECLARE @SzafkaID INT
		SELECT @SzafkaID = DELETED.ID_Szafki FROM DELETED

		UPDATE Szafki SET [Status] = 'wolna'
		WHERE ID_Szafki IN (@SzafkaID)
	END
```

```SQL
IF OBJECT_ID('Zmiana_wynagrodzenia', 'TR') IS NOT NULL
    DROP TRIGGER Zmiana_wynagrodzenia
GO
CREATE TRIGGER Zmiana_wynagrodzenia ON Pracownicy
	INSTEAD OF UPDATE
	AS
	BEGIN
		DECLARE @NoweWynagrodzenie INT, @StareWynagrodzenie INT, @PracownikID INT
		
		SELECT @NoweWynagrodzenie = INSERTED.Wynagrodzenie FROM INSERTED
		SELECT @StareWynagrodzenie = DELETED.Wynagrodzenie FROM DELETED
		SELECT @PracownikID = DELETED.ID_Pracownika FROM DELETED

		IF(@NoweWynagrodzenie > @StareWynagrodzenie)
			UPDATE Pracownicy SET Wynagrodzenie = @NoweWynagrodzenie WHERE ID_Pracownika = @PracownikID
		ELSE
			PRINT('Nie można obniżyć wynagrodzenia!')
	END

```
