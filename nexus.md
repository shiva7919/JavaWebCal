
-----

## 🌐 Automated Deployment System: Code to Web with 3 Servers

This document details the automated system you've built using three dedicated Linux servers to handle the **building, storing, and running** of your Java Web Calculator application. This setup creates a clear, repeatable process for pushing updates.

## 🧩 System Architecture Overview

The application follows a simple, one-way flow across the three machines.

| Server | Role | Primary Tool | Key Action |
| :--- | :--- | :--- | :--- |
| **Build Server** | Compiles the code. | Maven | Creates the `.war` file. |
| **Repository Server** | Stores versioned apps. | Nexus | Saves the `.war` file. |
| **Deploy Server** | Runs the live app. | Tomcat | Fetches and runs the `.war` file. |

```text
+----------------+       +------------------+       +-----------------+
|  Build Server  |----->| Nexus Repository |----->|  Deploy Server  |
| (Maven & JDK)  |       |  (Artifact Hub)  |       | (Tomcat & JRE)  |
+----------------+       +------------------+       +-----------------+
        |                        |                             |
  1. git clone             3. Stores .war file           4. Fetches .war file
  2. mvn deploy            (e.g., webapp-v0.1.war)       5. Deploys to Tomcat
```

-----

## 🏗️ Stage 1: The **Build Server** (The Code Workshop)

**Hostname:** `Build`

This machine's purpose is to take the Java source code and convert it into a final, ready-to-run file (`.war`).

### Setup and Process

1.  **Install Java and Maven:** The environment is set up by installing the necessary tools:

      * `sudo apt install openjdk-17-jre-headless -y`: Installs the **Java Runtime Environment (JRE)**, which is needed to run the build tool.
      * `sudo apt install maven -y`: Installs **Apache Maven**, the automation tool responsible for compiling the code.

2.  **Retrieve Source Code:** The calculator's blueprints are pulled from GitHub:

      * `git clone https://github.com/mrtechreddy/Java-Web-Calculator-App.git`

3.  **Configure for Nexus Upload:** This step tells Maven where the Nexus storage is and how to log in:

      * **`pom.xml` (Project Configuration):** The `<distributionManagement>` section is added to specify the **URL** of the target Nexus repository (`http://<NEXUS_IP>:8081/repository/java-cal-app/`).
      * **`settings.xml` (User Credentials):** The `<servers>` section is edited to securely store the **admin username and password** (`admin`/`admin123`) that Maven uses to authenticate with Nexus.

4.  **Build and Ship the App:**

      * `mvn package`: Compiles the Java code, runs tests, and creates the final **`.war` file**.
      * `mvn deploy`: Reads the credentials and URL, then **uploads** the finished `.war` file to the Nexus Repository.

-----

## 📦 Stage 2: The **Repository Server** (Central Storage)

**Hostname:** `Nexus-Repo`

This machine acts as the secured, organized warehouse, centralizing all versions of your built application files.

### Setup and Process

1.  **Nexus Installation and Start:** The Nexus software is downloaded, extracted, and launched as a service:

      * `wget ...nexus-3.85.0-03-linux-x86_64.tar.gz`
      * `tar -xvzf ...`
      * `./nexus start`

2.  **Web Access and Admin Setup:**

      * Nexus is accessed in a browser on port **8081** (`http://<NEXUS_PUBLIC_IP>:8081`).
      * The initial admin password must be retrieved from the server using `sudo cat sonatype-work/nexus3/admin.password`.

3.  **Create the Repository:** **This is the key step.** Within the Nexus web interface, a **new Hosted Repository** is created and named **`java-cal-app`**. This folder is the exact destination specified in the Build Server's `pom.xml` and is where the built `.war` file will be stored.

-----

## 🚀 Stage 3: The **Deploy Server** (The Live Environment)

**Hostname:** `Deploy`

This machine's function is to run the application for end-users by serving it over the web.

### Setup and Process

1.  **Install Prerequisites:**

      * `sudo apt install openjdk-17-jre-headless -y`: **Java JRE** is required, as the Tomcat server is a Java application.
      * **Tomcat Installation:** The Apache Tomcat server is downloaded and extracted.

2.  **Fetch the App from Nexus:** The application is pulled directly from the storage server using an authenticated download.

      * The **`wget`** command is used to download a **specific version** of the `.war` file from Nexus, using the admin credentials to access the secure path. The path is structured by Maven coordinates (e.g., `.../webapp-add/0.1/webapp-add-0.1.war`).
      * The downloaded `.war` file is placed into Tomcat's deployment folder, **`webapps/`**.

    *Example Command (fetching version 0.1):*

    ```bash
    wget http://admin:admin123@<NEXUS_IP>:8081/repository/java-cal-app/com/web/cal/webapp-add/0.1/webapp-add-0.1.war
    ```

3.  **Start Tomcat:**

      * Navigate to the Tomcat binaries: `cd apache-tomcat-9.0.111/bin/`
      * Run `./startup.sh` to start the web server. Tomcat automatically detects and deploys the `.war` file in its `webapps/` directory.

### ✅ Verification

  * **Tomcat Status:** Confirm the web server is running at `http://<DEPLOY_PUBLIC_IP>:8080`.
  * **Application Access:** The calculator app is now live and accessible at a URL based on the `.war` filename: `http://<DEPLOY_PUBLIC_IP>:8080/webapp-add-0.1/`.
---
## screenshots

1. <img width="1917" height="1079" alt="Image" src="https://github.com/user-attachments/assets/5553d78e-87ff-4cde-97c2-575117d778e9" />
2. <img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/33103acf-999a-475c-a4c6-11696fcfa762" />
3. <img width="1072" height="228" alt="Image" src="https://github.com/user-attachments/assets/452f31d9-3aba-4737-828f-2b5a45a07a80" />
4. <img width="1011" height="322" alt="Image" src="https://github.com/user-attachments/assets/b8a24234-8463-45d1-a72d-fe60d8fc72b7" />
5. <img width="1906" height="413" alt="Image" src="https://github.com/user-attachments/assets/8f1da6fa-4c2d-49d3-b416-7ba1c0c8ef88" />
6. <img width="1121" height="109" alt="Image" src="https://github.com/user-attachments/assets/ebf522d0-f957-4e1c-8fac-942bfeaabb9d" />
7. <img width="1919" height="540" alt="Image" src="https://github.com/user-attachments/assets/14549a60-7557-4344-ac65-3609e2f877a7" />
8. <img width="1919" height="1076" alt="Image" src="https://github.com/user-attachments/assets/6a8b989d-a35d-41ee-92cd-2601b6115fdf" />
9. <img width="1913" height="1079" alt="Image" src="https://github.com/user-attachments/assets/94c7aaa1-24db-4adf-99df-8fc35102515d" />
10. <img width="965" height="129" alt="Image" src="https://github.com/user-attachments/assets/9cfd6fa2-38da-4083-a1d3-8c14857aced8" />
11. <img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/587d960f-ddd8-4768-8c86-f791904d37c3" />
12. <img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/30e5c45f-18ca-436a-bed6-0bdc112ddc7c" />
13. <img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/6b25ef3f-abed-423f-9dcc-307c313d3aa9" />
14. <img width="1918" height="1069" alt="Image" src="https://github.com/user-attachments/assets/775145f6-1a94-44c8-8816-f2ce4496c6b3" />
15. <img width="1893" height="1079" alt="Image" src="https://github.com/user-attachments/assets/dbca295f-f0e8-40c5-9805-da7a9deced82" />
16. <img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/5b71fa26-8c7a-4d12-98dd-fc24b583dad9" />
17. <img width="1911" height="1079" alt="Image" src="https://github.com/user-attachments/assets/c138c916-8e20-4882-81aa-530de0c8bd0c" />
18. <img width="1148" height="382" alt="Image" src="https://github.com/user-attachments/assets/2c68a614-3ffd-4332-9b6f-6521b53d2300" />
19. <img width="772" height="347" alt="Image" src="https://github.com/user-attachments/assets/ab653cca-c1c0-4d3e-9a99-177b1e4ab7bf" />
20. <img width="1796" height="491" alt="Image" src="https://github.com/user-attachments/assets/81995458-a1a4-4ba0-90f9-8e71388460e4" />
21. <img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/b3c8f61e-76a4-4723-bbc5-836329d9f89e" />
22. <img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/777b617e-e67b-43c8-9b1b-05d73e748c18" />
23. <img width="1919" height="1079" alt="Image" src="https://github.com/user-attachments/assets/021a60b3-dcba-4998-a43c-3b480239b9c0" />
24.  <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/16361cda-01b2-46bc-849a-d5a891f6a3e2" />
25.  <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/f2dcfc41-1463-4348-83d2-c7b53f173542" />
26. <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/89a84f90-c972-48da-83c5-9f5012b2f5a1" />
27.  <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/09d09364-7e04-4db2-a6bd-27a2c8c2925d" />
28.  <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/526b87e5-bfe0-4d5b-bad1-f99c63eadf4c" />
    
 
