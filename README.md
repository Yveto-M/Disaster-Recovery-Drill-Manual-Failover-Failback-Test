# 📉 Disaster Recovery Drill: Manual Failover & Failback Test

**Version:** CyberArk PAS 13.2

**Scenario:** Unplanned outage of Primary Vault (`10.0.0.1`) requiring manual promotion of DR Site (`10.0.0.4`).

---

## Phase 1: Pre-Check (Baseline Health)
Before initiating the drill, I verified the DR Vault configuration and ensured replication was healthy.

1.  **DR Configuration Check:** Verified `EnableFailover` and `FailoverMode` settings.
    > **PADR Config Check**
    > <img width="808" height="198" alt="DR-PADR-check-failover-1" src="https://github.com/user-attachments/assets/310bc8ee-c803-422d-a59a-d79b45f86c31" />


2.  **Service Status:** Confirmed PrivateArk Database was active and replicating.
    > **DB Active**
    > <img width="802" height="268" alt="DR-DB-Active-prior-to-failover-drill-2" src="https://github.com/user-attachments/assets/cf273bbc-de1e-4cce-95e7-bc22dc21e059" />


3.  **Replication Verification:** `PADR.log` confirmed the database was in sync (`Replicate ended`).
    > **Healthy Replication**
    > <img width="810" height="171" alt="Dr-replication-completed-prior-to-failover-3" src="https://github.com/user-attachments/assets/a571bee6-094a-4274-b52a-c8c92d619997" />


---

## Phase 2: The Crash (Simulation)
**Action:** Executed a "Hard Stop" of the Primary Vault VM to simulate catastrophic hardware failure.

1.  **Forcing the Outage:**
    > **Simulated Crash**
    > <img width="551" height="252" alt="Pro-vault-shutdown-4" src="https://github.com/user-attachments/assets/31578afe-1087-4a22-9c12-ae2e7a941bdb" />


2.  **Impact Observation:** PVWA immediately became unavailable.
    > **PVWA Down**
    ><img width="813" height="648" alt="PVWA-server-unavailable-5" src="https://github.com/user-attachments/assets/9d0ca402-5fac-4030-8744-b29f637a7561" />


3.  **DR Log Verification:** The DR service logged errors confirming it could not contact the Primary Vault.
    > **PADR Failure Log**
    > <img width="849" height="175" alt="Dr-padr-log-fail-6" src="https://github.com/user-attachments/assets/af7aa3c1-fb3c-42d2-af35-e254c285f8f7" />

---

## Phase 3: The Failover (Manual Promotion)
**Objective:** Promote the DR Node (`10.0.0.4`) to be the Active Primary Vault.

1.  **Service Transition:** Stopped the DR Service and manually started the `PrivateArk Server`.
    > **Starting Vault Service**
    > <img width="804" height="132" alt="privateark-server-Dr-7" src="https://github.com/user-attachments/assets/137b7d4a-ac3b-4585-aa1c-f2d1047ce0e8" />


2.  **Vault Activation:** Verified the Vault Console (Server Central Administration) was up and running on the DR node.
    > **DR Vault Console Up**
    > <img width="880" height="318" alt="Dr-server-up-8" src="https://github.com/user-attachments/assets/4a08b93e-b222-4830-af6f-95f4af03a4e7" />


3.  **Traffic Routing:** Updated `Vault.ini` on the Component Server to include the DR IP address.
    > **Vault.ini Update**
    > <img width="754" height="273" alt="Vault in-Ip-update-9" src="https://github.com/user-attachments/assets/2e39be8b-bc90-4943-9123-5348f04ac706" />


4.  **Restarting Components:** Performed an `iisreset` to force the PVWA to read the new configuration.
    > **IIS Reset**
    > <img width="413" height="188" alt="iireset-10" src="https://github.com/user-attachments/assets/a043dc3d-c271-4289-8a97-b79d5cdab8f2" />


### 🚀 Proof of Failover
The System Health Dashboard confirmed that the **DR Node (`10.0.0.4`)** was now operating as the Primary Vault.
> **Proof of Failover**
> <img width="802" height="372" alt="proof-of-failover-11" src="https://github.com/user-attachments/assets/917fd7a0-5b86-47ee-8622-293ef82957be" />


---

## Phase 4: The Failback (Restoring Normal Operations)
**Objective:** Bring the original Primary Vault (`10.0.0.1`) back online and demote the DR Vault to passive status.

1.  **Primary Restoration:** Powered on the Primary Vault and verified it was active.
    > **Primary Vault Online**
    > <img width="960" height="290" alt="Failover-production-vault-up-13" src="https://github.com/user-attachments/assets/325158b7-261b-4c28-b876-81678a49f127" />


2.  **Demoting DR Vault:** Manually stopped the `PrivateArk Server` on the DR node to prevent a "Split-Brain" scenario.
    > **Stopping DR Vault**
    > <img width="597" height="341" alt="failback-privateserver-stop-12" src="https://github.com/user-attachments/assets/c38968a4-3d02-41ff-bf8f-4d133d01ee3b" />


3.  **Configuring for Passive Mode:** Edited `PADR.ini` to ensure `FailoverMode=No`.
    > **PADR Config Reset**
    > <img width="625" height="192" alt="Dr-PADR-failover-no-14" src="https://github.com/user-attachments/assets/cd2ae236-7232-42c5-b02e-e2c151ee796c" />


4.  **Restarting Replication:** Started the `CyberArk Disaster Recovery Service`.
    > **Starting DR Service**
    > <img width="662" height="103" alt="DR-DR-start-15" src="https://github.com/user-attachments/assets/960629cb-653a-4368-a59c-cf0a1668538d" />


---

## Phase 5: Troubleshooting (The "Zombie" Incident)
During the failback, the replication initially failed. This section documents the troubleshooting steps.

1.  **The Error:** The DR Service attempted to start the Vault instead of replicating, throwing error `PADR0013E`.
    > **Split Brain Error**
    > <img width="832" height="129" alt="Screenshot 2025-12-28 211440" src="https://github.com/user-attachments/assets/fbb925a5-5ebd-45e7-a476-2cb2bd98736e" />


2.  **Root Cause Analysis:** A "Zombie" process (`CyberArk Logic Container`) was holding a lock on the database. The Console showed "Terminated," but the Service showed "Running."
    > **Service Conflict**
    > <img width="886" height="404" alt="Screenshot 2025-12-28 212927" src="https://github.com/user-attachments/assets/639231e7-536b-4e04-a0b6-0c3c686cfac8" />


3.  **The Fix:** Killed the process and edited `PADR.ini` to force a full re-sync by deleting the timestamp.
    > **Config Fix**
    > <img width="846" height="366" alt="Screenshot 2025-12-28 214113" src="https://github.com/user-attachments/assets/7772e2f6-9f24-4722-a8a2-33bf6b51f09a" />


---

## Phase 6: Final Success
After applying the fix, the system triggered a Full Replication.

1.  **Full Sync Triggered:**
    > **Full Sync Start**
    > <img width="673" height="114" alt="Screenshot 2025-12-28 214955" src="https://github.com/user-attachments/assets/1c959e83-91a9-49ae-93b0-7abae18b0732" />


2.  **Replication Complete:** The system returned to a healthy, redundant state.
    > **Success Log**
    > <img width="849" height="117" alt="Money-shot-failover-complete-16" src="https://github.com/user-attachments/assets/95ec3a1a-f815-4635-95f2-771bc699616a" />
