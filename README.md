# Sprawozdanie - Zadanie 2

## 1. Architektura i Konfiguracja Etapów Łańcucha

**a. Wieloplatformowość obrazu**
Obraz wspiera architektury linux/arm64 oraz linux/amd64. Wykorzystano akcje:
* `docker/setup-qemu-action` konfigurującą emulację sprzętową QEMU.
* `docker/setup-buildx-action` uruchamiającą silnik kompilacji wieloarchitektonicznej.
* W kroku budowania zadeklarowano parametr docelowy: platforms: linux/amd64,linux/arm64.

**b. Konfiguracja pamięci podręcznej (cache)**
Wdrożono mechanizm zarządzania pamięcią podręczną oparty na zewnętrznym rejestrze na platformie DockerHub. Użyto eksportera type=registry. Eksport obejmuje wszystkie warstwy obrazu dzięki przypisaniu atrybutu mode=max. Uwierzytelnienie zrealizowano przy użyciu zmiennej vars.DOCKERHUB_USERNAME oraz sekretu secrets.DOCKERHUB_TOKEN.

**c. Analiza CVE**
Zastosowano skaner Trivy (aquasecurity/trivy-action). Wygenerowano obraz lokalny (load: true), na którym skaner analizuje system i biblioteki (vuln-type: 'os,library'). Blokadę eksportu do repozytorium na platformie GitHub w przypadku wykrycia zagrożeń krytycznych i wysokich uzyskano poprzez konfigurację severity: 'CRITICAL,HIGH' oraz wymuszenie kodu błędu exit-code: '1'.

<img width="298" height="185" alt="Zrzut ekranu 2026-06-06 222915" src="https://github.com/user-attachments/assets/ee5fe21b-2517-49ed-83a0-86fbe268a81d" />


## 2. Przyjęty sposób tagowania obrazów i danych cache

**a. Tagowanie obrazów aplikacji (GitHub Container Registry - ghcr.io)**
Wdrożono strategię opartą na akcji docker/metadata-action. Tagowanie sumą kontrolną commita type=sha,format=short otrzymało priorytet 100. Tagowanie Semantic Versioning type=semver,pattern={{version}} otrzymało priorytet 200. Domyślny tag latest wyłączono parametrem flavor: latest=false.
*Uzasadnienie:* Wyłączenie znacznika latest eliminuje modyfikację istniejących obrazów. Skrócony skrót SHA z priorytetem 100 zapewnia identyfikację rewizji kodu (wzorzec GitOps). Schemat semver z priorytetem 200 w pełni nadpisuje domyślny znacznik podczas wyzwalania łańcucha nowym tagiem wydania.

**b. Tagowanie danych cache (DockerHub)**
Dla warstw pośrednich wyeksportowanych na platformę DockerHub przypisano stały znacznik :cache.
*Uzasadnienie:* Parametr mode=max w silniku Buildkit pozwala na eksport wszystkich warstw budowania. Silnik Buildx nadpisuje wygasłe bloki podczas iteracji. Utrzymanie pojedynczego, stałego znacznika zapobiega gwałtownemu zużyciu przestrzeni magazynowej w rejestrze zewnętrznym i zapewnia nadpisywanie zmodyfikowanych fragmentów cache w obrębie danego wpisu.


## 3. Potwierdzenie działania łańcucha

<img width="1340" height="573" alt="Zrzut ekranu 2026-06-06 222733" src="https://github.com/user-attachments/assets/56f10b61-1de3-46b3-8fbc-63dba02d853a" />
