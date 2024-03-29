# Vault-KV-V2
<br>

## Summary
**In this lab, we will test below scenerios step by step.**

**- vault secrets enable & write & read** <br>
**- vault kv put & get** <br>
**- vault kv delete** <br> 
**- vault kv destroy** <br>
**- vault kv destroy** <br>
**- vault kv metadata** <br>
**- delete_version_after** <br>
<br>

### 1) How to enable & write & read vault secrets 

**Enable secret using Path & versoins**
- Enable kv v2 secrets engine at DB-Team/ path.
- Enable kv v1 secrets engine at  DB-Team2/ path.
<br>

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault secrets enable --version=2 --path=DB-Team kv
Success! Enabled the kv secrets engine at: DB-Team/

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault secrets enable  --path=DB-Team2 kv
Success! Enabled the kv secrets engine at: DB-Team2/

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
DB-Team/      kv           kv_4d08ff18           n/a
DB-Team2/     kv           kv_3ad8df39           n/a
cubbyhole/    cubbyhole    cubbyhole_63a670bf    per-token private secret storage
identity/     identity     identity_bb56535d     identity store
secret/       kv           kv_966419c8           key/value secret storage
sys/          system       system_59797446       system endpoints used for control, policy and debugging
```

<br>

**We can upgrade V1 to V2 as below.**

<br>

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv enable-versioning DB-Team2/
Success! Tuned the secrets engine at: DB-Team2/

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault read DB-Team2/config
Key                     Value
---                     -----
cas_required            false
delete_version_after    0s
max_versions            0
```

<br>


cas - stands for Check-and-Set which is used to protect from being overwritten unintentionally. 

delete_version_after  is set to 0s which means Vault will delete versions after 0s.

max_versions is set 0 which means there is no limit on versions. All the versions will be kept. 

<br>

**We can limit the max version as below.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault write DB-Team2/config/ max_versions=5
Success! Data written to: DB-Team2/config
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault read DB-Team2/config
Key                     Value
---                     -----
cas_required            false
delete_version_after    0s
max_versions            5
```

<br>

### 2) vault kv put & get

**We can add secrets in DB-Team2/ as below.** 

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv put DB-Team2/members/James FullName="Jame Liew"
======= Secret Path =======
DB-Team2/data/members/James

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:26:12.400063067Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

```
<br>

**We can read secrets in DB-Team2/ as below**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  DB-Team2/members/James
======= Secret Path =======
DB-Team2/data/members/James

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:26:12.400063067Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

====== Data ======
Key         Value
---         -----
FullName    Jame Liew
```

**Let's put new secret date-info under DB-Team2**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv put DB-Team2/Data/dabe-info username="sa" password="fsjldkj"
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:27:31.73396955Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

```


**Then let's try to update the values in secret. <br>
Verify the version was changed to 2.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv put DB-Team2/Data/dabe-info username="sa" password="00002"
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:30:40.282220676Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            2
```

<br>

**Let's update till we see the version is 5.**

```yaml
vault kv put DB-Team2/Data/dabe-info username="sa" password="00002"
vault kv put DB-Team2/Data/dabe-info username="admin" password="11232"
vault kv put DB-Team2/Data/dabe-info username="admi1" password="11232"

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:37:37.011313668Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            5

====== Data ======
Key         Value
---         -----
password    11232
username    admi1
```

**Now we have 5 versions in DB-Team2/Data/dabe-info. <br>
We can read older versions as below.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=1 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:27:31.73396955Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

====== Data ======
Key         Value
---         -----
password    fsjldkj
username    sa

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=2 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:30:40.282220676Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            2

====== Data ======
Key         Value
---         -----
password    00002
username    sa

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=4 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:37:23.368283659Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            4

====== Data ======
Key         Value
---         -----
password    11232
username    admi1
```


**Let's update the values more time and see the oldest version was deleted.**

```yaml
vault kv put DB-Team2/Data/dabe-info username="admi1" password="dfkdfs"
vault kv put DB-Team2/Data/dabe-info username="admi1" password="dfkd"

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:43:30.57290747Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            7

====== Data ======
Key         Value
---         -----
password    dfkd
username    admi1
```
<br>

**We updated the values 2 times and now version 1&2 was deleted alread.**


```yaml

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=1 DB-Team2/Data/dabe-info
No value found at DB-Team2/data/Data/dabe-info
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=2 DB-Team2/Data/dabe-info
No value found at DB-Team2/data/Data/dabe-info
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=3 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:36:54.416417003Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            3

====== Data ======
Key         Value
---         -----
password    11232
username    admin
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$
```

![image](https://github.com/myathway-lab/3-Vault-KV-V2/assets/157335804/5ba45d66-ac99-49df-be75-1620415b53ff)

<br>


### 3) vault kv delete

**Delete the data in DB-Team2/data/Data/dabe-info.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv delete DB-Team2/Data/dabe-info
Success! Data deleted (if it existed) at: DB-Team2/data/Data/dabe-info
```
<br>

**We can see there is no more data in DB-Team2/Data/dabe-info.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:43:30.57290747Z
custom_metadata    <nil>
deletion_time      2024-02-27T16:52:48.989802168Z
destroyed          false
version            7
```

**Let’s verify from UI. <br>
We can see Version7 was deleted.**

![image](https://github.com/myathway-lab/3-Vault-KV-V2/assets/157335804/fe32828b-4f07-41e6-82e7-7d53c001d49f)

 <br>
 
**We can Undelete the version7 until we destroyed.**

![image](https://github.com/myathway-lab/3-Vault-KV-V2/assets/157335804/e6ae0e94-b89e-44d3-ba1b-25a681e5ea3a)

 <br>
 
**But We still can read the old versions metadata as below.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=6 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:42:14.973575277Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            6

====== Data ======
Key         Value
---         -----
password    dfkdfs
username    admi1
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=5 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:37:37.011313668Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            5

====== Data ======
Key         Value
---         -----
password    2222
username    admi1
```

<br>
 
**Now, let's update the data without destroying the version 7.***

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv put DB-Team2/Data/dabe-info username="admi1" password="dfkd"
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T17:02:04.867567704Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            8
```
 <br>
 
**We can see total 8 versions. version 7 was not replaced. It created version 8 for the new data.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=7 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:43:30.57290747Z
custom_metadata    <nil>
deletion_time      2024-02-27T16:52:48.989802168Z
destroyed          false
version            7

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=8 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T17:02:04.867567704Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            8

====== Data ======
Key         Value
---         -----
password    dfkd
username    admi1
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$
```
 <br>
 
### 4) vault kv destroy

**Destroy version 6 as below. It will be permanently deleted and cannot be restored.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=6 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:42:14.973575277Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            6

====== Data ======
Key         Value
---         -----
password    dfkdfs
username    admi1

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv destroy --versions=6 DB-Team2/Data/dabe-info
Success! Data written to: DB-Team2/destroy/Data/dabe-info

myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  --version=6 DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T16:42:14.973575277Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          true
version            6
```

![image](https://github.com/myathway-lab/3-Vault-KV-V2/assets/157335804/2b850879-ce6d-4f8f-823a-d665f0fa898a)

<br>

### 5) vault kv metadata

**Delete the secret DB-Team2/metadata/Data/dabe-info.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv metadata delete DB-Team2/Data/dabe-info
Success! Data deleted (if it existed) at: DB-Team2/metadata/Data/dabe-info
```

**All the data "dabe-info" gone.**

![image](https://github.com/myathway-lab/3-Vault-KV-V2/assets/157335804/e44aa41c-3c2a-4236-a029-225bd1573528)

<br>

### 6) delete_version_after

**Set automatic deletion of versions after the fixed time period.** <br>

**Now we will set 60s to delete data in DB-Team2/metadata/Data/dabe-info.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv metadata put --delete-version-after=60s DB-Team2/Data/dabe-info
Success! Data written to: DB-Team2/metadata/Data/dabe-info
```

**Now let’s put some data into dabe-info.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv put DB-Team2/Data/dabe-info username="admin" password="pass123"
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T17:22:46.974855961Z
custom_metadata    <nil>
deletion_time      2024-02-27T17:23:46.974855961Z
destroyed          false
version            2
```

**Monitor if the data was deleted after 60s.**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T17:22:46.974855961Z
custom_metadata    <nil>
deletion_time      2024-02-27T17:23:46.974855961Z
destroyed          false
version            2

====== Data ======
Key         Value
---         -----
password    pass123
username    admin
```

**After 60s.....**

```yaml
myathway@DESKTOP-QCOTPM5:/mnt/c/WINDOWS/system32$ vault kv get  DB-Team2/Data/dabe-info
======== Secret Path ========
DB-Team2/data/Data/dabe-info

======= Metadata =======
Key                Value
---                -----
created_time       2024-02-27T17:22:46.974855961Z
custom_metadata    <nil>
deletion_time      2024-02-27T17:23:46.974855961Z
destroyed          false
version            2
```


