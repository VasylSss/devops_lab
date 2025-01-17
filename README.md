# Sprawozdanie końcowe: Projekt na podstawie "Building a RESTful Web Service"

## Wstęp
Celem laboratorium w ramach zajęć **Cykl życia i narzędzia DevOps** było zrozumienie podstawowych zasad budowania aplikacji RESTful przy użyciu Spring Boot, a także praktyczne wykorzystanie narzędzi DevOps takich jak Linux, Git, Docker, Docker-Compose, GitHub Actions oraz Kubernetes. Projekt bazował na przewodniku dostępnym na stronie: [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service). 

Projekt ten umożliwił uczestnikom zdobycie doświadczenia w zarządzaniu cyklem życia aplikacji oraz ich infrastrukturą w nowoczesnym środowisku developerskim.

---

## Linux

Lista komend użytych podczas laboratorium:

1. **Sprawdź, w którym katalogu aktualnie się znajdujesz:**
   ```bash
   pwd
   ```
   Wyświetla ścieżkę do bieżącego katalogu roboczego.

2. **Przenieś się do katalogu domowego:**
   ```bash
   cd ~
   ```
   Przenosi użytkownika do katalogu domowego.

3. **Przejdź do katalogu /etc:**
   ```bash
   cd /etc
   ```
   Zmienia bieżący katalog roboczy na `/etc`.

4. **Utwórz nowy katalog o nazwie Lab1 w katalogu domowym:**
   ```bash
   mkdir ~/Lab1
   ```
   Tworzy katalog `Lab1` w katalogu domowym.

5. **Wewnątrz katalogu Lab1 stwórz plik tekstowy notatki.txt i edytuj go w vim:**
   ```bash
   cd ~/Lab1
   vim notatki.txt
   ```
   Tworzy plik `notatki.txt` i umożliwia jego edycję za pomocą edytora Vim.

6. **Wyświetl zawartość pliku notatki.txt:**
   ```bash
   cat notatki.txt
   ```
   Wyświetla zawartość pliku na ekranie.

7. **Sprawdź uprawnienia do pliku notatki.txt i zmień je:**
   ```bash
   ls -l notatki.txt
   chmod 777 notatki.txt
   ```
   Wyświetla uprawnienia, a następnie zmienia je, aby wszyscy użytkownicy mogli czytać i zapisywać plik.

8. **Przekieruj wynik polecenia ls /etc do pliku lista_plikow.txt:**
   ```bash
   ls /etc > lista_plikow.txt
   cat lista_plikow.txt
   ```
   Zapisuje listę plików z katalogu `/etc` do pliku `lista_plikow.txt` i wyświetla jego zawartość.

9. **Znajdź wszystkie pliki w katalogu /etc z rozszerzeniem .conf:**
   ```bash
   find /etc -name "*.conf"
   ```
   Wyszukuje pliki z rozszerzeniem `.conf` w katalogu `/etc`.

10. **Zainstaluj program htop i uruchom go:**
    ```bash
    sudo apt update
    sudo apt install htop
    htop
    ```
    Aktualizuje listę pakietów, instaluje `htop`, a następnie uruchamia ten program do monitorowania zasobów systemowych.

---

## Git

Lista wykorzystanych komend:

1. **Stworzenie nowego brancha:**
   ```bash
   git checkout -b nazwa_brancha
   ```
2. **Dodanie zmian do commit:**
   ```bash
   git add .
   git commit -m "Opis zmian"
   ```
3. **Wypchnięcie brancha na zdalne repozytorium:**
   ```bash
   git push origin nazwa_brancha
   ```

**Procedura generowania klucza SSH i dodawania go do GitHub:**
1. Generowanie klucza SSH:
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```
   Tworzy klucz SSH w domyślnym katalogu.

2. Dodanie klucza do agenta SSH:
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   ```

3. Skopiowanie klucza publicznego:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   Klucz ten należy wkleić w ustawieniach GitHub w sekcji SSH and GPG Keys.

**Dlaczego używamy kluczy SSH?**
Klucze SSH zapewniają bezpieczne połączenie między lokalnym systemem a zdalnym repozytorium, eliminując potrzebę podawania hasła przy każdym połączeniu.

---

## Docker

**Czym jest konteneryzacja?**
Konteneryzacja to technologia pozwalająca na izolację aplikacji w lekkich, przenośnych kontenerach, które zawierają wszystkie zależności potrzebne do uruchomienia aplikacji.

**Dockerfile:**
```Dockerfile
# Kontener z mavenem, który umożliwi zbudowanie aplikacji
FROM maven:3.8.8-eclipse-temurin-17 AS build

# Ustawianie workdira w kontenerze
WORKDIR /app

# Kopiowanie poma i plików projektu do kontenera
COPY pom.xml .
COPY src ./src

# Budowanie aplikacji (dla naszych celów możemy pominąć testy)
RUN mvn clean package -DskipTests

# Wybranie lekkiego kontenera linuxowego z openjdk17
FROM openjdk:17-jdk-slim

# Ustawianie workdira w kontenerze
WORKDIR /app

# Skopiowanie zbudowanej jarki z kontenera budującego do kontenera uruchamiającego aplikację
COPY --from=build /app/target/restservice-0.0.1-SNAPSHOT.jar /app/restservice.jar

# Wystawienie portu 8080
EXPOSE 8080

# Uruchomienie aplikacji przy starcie kontenera
ENTRYPOINT ["java", "-jar", "/app/restservice.jar"]
```
**Wyjaśnienie komend:**
- `FROM maven:3.8.8-eclipse-temurin-17 AS build` - Używa obrazu Maven do zbudowania aplikacji.
- `WORKDIR /app` - Ustawia katalog roboczy na `/app`.
- `COPY pom.xml .` - Kopiuje plik `pom.xml` do katalogu roboczego.
- `COPY src ./src` - Kopiuje katalog źródłowy projektu do kontenera.
- `RUN mvn clean package -DskipTests` - Buduje aplikację, pomijając testy.
- `FROM openjdk:17-jdk-slim` - Używa lekkiego obrazu OpenJDK do uruchomienia aplikacji.
- `WORKDIR /app` - Ustawia katalog roboczy na `/app`.
- `COPY --from=build /app/target/restservice-0.0.1-SNAPSHOT.jar /app/restservice.jar` - Kopiuje zbudowaną aplikację z etapu budowania.
- `EXPOSE 8080` - Otwiera port 8080 w kontenerze.
- `ENTRYPOINT` - Określa komendę uruchamianą przy starcie kontenera.

---

## Docker-Compose

**Czym jest Docker-Compose?**
Docker-Compose to narzędzie umożliwiające definiowanie i uruchamianie wielokontenerowych aplikacji za pomocą pliku YAML. Ułatwia zarządzanie środowiskami aplikacji.

**docker-compose.yml:**
```yaml
version: '3'
services:
  mysql-db:
    image: mysql:8.0
    container_name: mysql-db
    environment:
      - MYSQL_ROOT_PASSWORD=rootpassword
      - MYSQL_DATABASE=mydb
    ports:
      - "3306:3306"
    networks:
      - my-network
    volumes:
      - mysql-data:/var/lib/mysql

  spring-boot-app:
    build: .
    container_name: spring-boot-app
    ports:
      - "8080:8080"
    environment:
      - JAVA_OPTS=
    depends_on:
      - mysql-db
    networks:
      - my-network

networks:
  my-network:

volumes:
  mysql-data:
```

**Wyjaśnienie sekcji:**
- `version` - Wskazuje wersję składni Docker-Compose.
- `services` - Definiuje usługi wchodzące w skład aplikacji.
- `mysql-db` - Konfiguracja bazy danych MySQL, z określeniem obrazu, portów, zmiennych środowiskowych i sieci.
- `spring-boot-app` - Konfiguracja aplikacji Spring Boot, która buduje obraz na podstawie lokalnego Dockerfile.
- `networks` - Tworzy dedykowaną sieć dla komunikacji między usługami.
- `volumes` - Tworzy wolumeny do przechowywania danych MySQL.

---

## Continuous Integration (CI)

**Czym jest CI?**
Continuous Integration to praktyka polegająca na regularnym integrowaniu zmian w kodzie do głównej gałęzi, z automatycznym uruchamianiem testów i budowaniem projektu.

**GitHub Actions Workflow:**
```yaml
name: Java CI with Maven

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven
    - name: Build with Maven
      run: mvn -B package --file pom.xml
```
**Wyjaśnienie:**
- `on` - Określa zdarzenie wyzwalające workflow (push na branch main).
- `jobs` - Definiuje zadania workflow.
- `steps` - Lista kroków, np. checkout kodu, ustawienie JDK, uruchomienie komend Maven.

---

## Kubernetes

**Po co stosujemy Kubernetesa?**
Kubernetes to system zarządzania kontenerami, który automatyzuje ich wdrażanie, skalowanie i zarządzanie. Ułatwia obsługę aplikacji w środowiskach chmurowych.

**Plik konfiguracyjny:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-app
spec:
  selector:
    matchLabels:
      app: spring-boot-app
  replicas: 5
  template:
    metadata:
      labels:
        app: spring-boot-app
    spec:
      containers:
        - name: spring-boot-app
          image: your-spring-boot-image
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
          env:
            - name: JAVA_OPTS
              value: ""
---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-service
spec:
  ports:
    - port: 8080
      targetPort: 8080
  selector:
    app: spring-boot-app
  type: NodePort
```
**Wyjaśnienie:**
- `Deployment` - Definiuje wdrożenie aplikacji, w tym liczbę replik ustawioną na 3.
- `Service` - Umożliwia dostęp do aplikacji poprzez LoadBalancer.

## Wnioski

Podczas realizacji projektu w ramach laboratorium Cykl życia i narzędzia DevOps, zdobyto cenne doświadczenie w projektowaniu, implementacji oraz zarządzaniu nowoczesnymi aplikacjami webowymi.

Kluczowe wnioski:

### Zastosowanie narzędzi DevOps:
Narzędzia takie jak Docker, Docker-Compose, GitHub Actions czy Kubernetes znacząco upraszczają zarządzanie cyklem życia aplikacji. Ich wykorzystanie pozwala na automatyzację wielu procesów, co zwiększa efektywność pracy zespołu developerskiego.

### Znaczenie CI/CD:
Continuous Integration i Continuous Deployment umożliwiają szybkie dostarczanie nowych funkcjonalności do produkcji, minimalizując ryzyko błędów dzięki automatycznym testom i procesom budowania aplikacji.

### Konteneryzacja i orkiestracja:
Dzięki Dockerowi i Kubernetesowi możliwa jest łatwa skalowalność aplikacji, zarządzanie jej komponentami oraz szybka adaptacja do zmieniających się wymagań środowiska.

### Podstawowa znajomość Linuxa:
Opanowanie podstawowych komend Linuxa jest kluczowe dla pracy w środowiskach serwerowych i kontenerowych, które często bazują na dystrybucjach tego systemu operacyjnego.

### Zarządzanie wersjami za pomocą Gita:
Praca z systemem kontroli wersji pozwala na efektywną współpracę zespołu, kontrolę historii zmian oraz łatwe wdrażanie poprawek czy nowych funkcji.

**Obszary do dalszego rozwoju:**

Testowanie i monitoring: Wprowadzenie zaawansowanych strategii testowania i narzędzi do monitoringu aplikacji zwiększyłoby stabilność i jakość dostarczanych rozwiązań.

Optymalizacja procesów CI/CD: Można rozszerzyć pipeline o kroki takie jak automatyczne testy integracyjne czy wdrożenia na środowisko stagingowe.

Podsumowując, realizacja projektu pozwoliła na zdobycie praktycznych umiejętności w zakresie wdrażania nowoczesnych aplikacji w środowisku produkcyjnym oraz zrozumienie kluczowych aspektów pracy w modelu DevOps.
