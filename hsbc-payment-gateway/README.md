# Nexus-login-demo

A minimal Spring Boot app that serves a small frontend (Thymeleaf) with form login (Spring Security). It simulates a simple bank "Payment Portal" with a public landing page and a protected dashboard, and is wired up for static analysis (SonarQube) and artifact deployment (Nexus).

## Tech stack

- Java 17
- Spring Boot 3.5 (Web, Thymeleaf, Security)
- Maven (build & dependency management)
- SonarQube (static code analysis)
- Nexus Repository (artifact hosting/deployment)

## Project structure
```
nexus-login-demo/
├── src/
│   ├── main/
│   │   ├── java/com/example/demo/
│   │   │   ├── DemoApplication.java         # Spring Boot entry point
│   │   │   ├── web/HomeController.java      # Routes: "/", "/public", "/admin"
│   │   │   └── security/SecurityConfig.java # Form login + in-memory users
│   │   └── resources/
│   │       ├── templates/                   # index.html, login.html, public.html, admin.html
│   │       ├── static/css/app.css
│   │       └── application.yml
│   └── test/java/com/example/demo/
│       └── AuthFlowTest.java                # MockMvc auth flow tests
├── pom.xml
└── .mvn/settings.xml
```
## Features

- **Public landing page** (`/public`) — accessible without logging in.
- **Protected dashboard** (`/`) — requires authentication; shows a personalized welcome and demo payment data.
- **Admin-only page** (`/admin`) — requires the `ADMIN` role; demonstrates role-based access control.
- **Form login** (`/login`) — Spring Security login form with logout support.
- **In-memory user store** — a regular user and an admin user, credentials externalized to `application.yml`, passwords encoded with BCrypt.
- **Unit tests** — verify redirect-to-login, public page access, and authenticated access using MockMvc.

## Routes

| Route     | Access            | Description                       |
|-----------|-------------------|------------------------------------|
| `/public` | Public            | Marketing/landing page            |
| `/`       | Authenticated     | Protected dashboard (payment portal) |
| `/admin`  | Role: `ADMIN`     | Admin-only page                   |
| `/login`  | Public            | Login form                        |
| `/logout` | Authenticated     | Logs the user out, redirects to `/public` |

## Run locally

```bash
mvn spring-boot:run
```

Open:
- http://localhost:8080/public (no login)
- http://localhost:8080/ (requires login)
- http://localhost:8080/admin (requires the admin login)

Demo credentials:
- Regular user: `user / password`
- Admin user: `admin / adminpass`

## Unit tests

```bash
mvn test
```

## Static code analysis with SonarQube

```bash
mvn -DskipTests sonar:sonar \
  -Dsonar.host.url=https://YOUR_SONAR_HOST \
  -Dsonar.token=YOUR_SONAR_TOKEN
```

## Running on an EC2 instance

These steps walk through provisioning a fresh EC2 instance, cloning this repo, and running it through the full Maven build lifecycle (`validate` → `compile` → `test` → `package` → `verify` → `install` → `deploy`).

### 1) Connect to the instance

```bash
ssh -i your-key.pem ec2-user@<EC2_PUBLIC_IP>
```

(Use `ubuntu` instead of `ec2-user` if you launched an Ubuntu AMI.)

### 2) Install Git and clone the repo

```bash
sudo yum install -y git      # Amazon Linux / RHEL
# or
sudo apt update && sudo apt install -y git   # Ubuntu/Debian

git clone <YOUR_REPO_URL>.git
cd nexus-login-demo
```

### 3) Install Java 17 and Maven

```bash
sudo yum install -y java-17-amazon-corretto maven   # Amazon Linux
# or
sudo apt install -y openjdk-17-jdk maven             # Ubuntu/Debian

java -version
mvn -version
```

### 4) Install `tree` to visualize the project structure

```bash
sudo yum install -y tree   # Amazon Linux / RHEL
# or
sudo apt install -y tree   # Ubuntu/Debian

tree -L 2
```

This prints the project's directory layout (two levels deep) so you can confirm `src/`, `pom.xml`, and other key files cloned correctly before building.

### 5) Walk through the Maven build lifecycle

Run each phase individually to see the lifecycle in action — Maven automatically runs all preceding phases when you call a later one:

```bash
mvn validate   # checks the project is correct and all necessary info is available
mvn compile    # compiles the source code
mvn test       # runs unit tests using a suitable testing framework
mvn package    # packages compiled code into its distributable format (.jar)
mvn verify     # runs checks to verify the package is valid and meets quality criteria
mvn install    # installs the package into the local repo (~/.m2), for use as a dependency locally
mvn deploy     # copies the final package to the remote repository (Nexus), for sharing with other developers
```

Or run the sequence up to `package` in one go:

```bash
mvn clean package
```

### 6) Run the app on EC2

```bash
mvn spring-boot:run
```

Or run the packaged jar directly:

```bash
java -jar target/*.jar
```

Make sure the EC2 instance's security group allows inbound traffic on port `8080`, then visit:
- `http://<EC2_PUBLIC_IP>:8080/public` (no login)
- `http://<EC2_PUBLIC_IP>:8080/` (requires login)

## "Dry build with Nexus" (resolve dependencies via Nexus mirror)

This uses `.mvn/settings.xml` which mirrors all Maven downloads through Nexus.

```bash
mvn -s .mvn/settings.xml -Pdry-nexus clean verify
```

## Deploy artifacts to Nexus

1) Set environment variables used by `.mvn/settings.xml`:

```bash
export NEXUS_USERNAME=...
export NEXUS_PASSWORD=...
```

2) Update `pom.xml`:
- `distributionManagement` URLs
- `nexusUrl` and `repository` inside the `nexus-deploy` profile

3) Deploy:

```bash
mvn -s .mvn/settings.xml -Pnexus-deploy clean deploy
```
