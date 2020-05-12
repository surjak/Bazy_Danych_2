
## ETCD LAB 

A distributed, reliable key-value store for the most critical data of a distributed system.  
Homepage: https://etcd.io/

Key features:

- Simple: well-defined, user-facing API (gRPC)
- Secure: automatic TLS with optional client cert authentication
- Fast: benchmarked 10,000 writes/sec
- Reliable: properly distributed using Raft


There are two major use cases: concurrency control in the distributed system and application configuration store. For example, CoreOS Container Linux uses etcd to achieve a global semaphore to avoid that all nodes in the cluster rebooting at the same time. Also, Kubernetes use etcd for their configuration store.

During this lab we will be using etcd3 python client.  
Homepage: https://pypi.org/project/etcd3/

Please copy & paste them into the cell below:


```python
etcdCreds = {
# paste your credentials
}
```


```python
!pip install etcd3
```

    Requirement already satisfied: etcd3 in /opt/conda/envs/Python36/lib/python3.6/site-packages (0.12.0)
    Requirement already satisfied: tenacity>=6.1.0 in /opt/conda/envs/Python36/lib/python3.6/site-packages (from etcd3) (6.2.0)
    Requirement already satisfied: grpcio>=1.27.1 in /opt/conda/envs/Python36/lib/python3.6/site-packages (from etcd3) (1.28.1)
    Requirement already satisfied: six>=1.12.0 in /opt/conda/envs/Python36/lib/python3.6/site-packages (from etcd3) (1.12.0)
    Requirement already satisfied: protobuf>=3.6.1 in /opt/conda/envs/Python36/lib/python3.6/site-packages (from etcd3) (3.6.1)
    Requirement already satisfied: setuptools in /opt/conda/envs/Python36/lib/python3.6/site-packages (from protobuf>=3.6.1->etcd3) (40.8.0)


### How to connect to etcd using certyficate (part 1: prepare file with certificate)


```python
import base64
import tempfile

etcdHost = etcdCreds["connection"]["grpc"]["hosts"][0]["hostname"]
etcdPort = etcdCreds["connection"]["grpc"]["hosts"][0]["port"]
etcdUser = etcdCreds["connection"]["grpc"]["authentication"]["username"]
etcdPasswd = etcdCreds["connection"]["grpc"]["authentication"]["password"]
etcdCertBase64 = etcdCreds["connection"]["grpc"]["certificate"]["certificate_base64"]
                           
etcdCertDecoded = base64.b64decode(etcdCertBase64)
etcdCertPath = "{}/{}.cert".format(tempfile.gettempdir(), etcdUser)
                           
with open(etcdCertPath, 'wb') as f:
    f.write(etcdCertDecoded)

print(etcdCertPath)
```

    /home/dsxuser/.tmp/ibm_cloud_f59f3a7b_7578_4cf8_ba20_6df3b352ab46.cert


### Short Lab description

During the lab we will simulate system that keeps track of logged users
- All users will be stored under parent key (path): /logged_users
- Each user will be represented by key value pair
    - key /logged_users/name_of_the_user
    - value hostname of the machine (e.g. name_of_the_user-hostname)

### How to connect to etcd using certyficate (part 2: create client)


```python
import etcd3

etcd = etcd3.client(
    host=etcdHost,
    port=etcdPort,
    user=etcdUser,
    password=etcdPasswd,
    ca_cert=etcdCertPath
)

cfgRoot='/logged_users'
```

### Task1 : Fetch username and hostname

define two variables
- username name of the logged user (tip: use getpass library)
- hostname hostname of your mcomputer (tip: use socket library)


```python
import getpass
import socket

# username = getpass.getuser()  # You can put your name here, while this code is run in the container and user name would be same for all students
username = "surjak"
hostname = socket.gethostname()

userKey='{}/{}'.format(cfgRoot, username)
userKey, '->', hostname
etcd.put(userKey, hostname)
```




    header {
      cluster_id: 17394822126184162018
      member_id: 9522968931340837552
      revision: 315572
      raft_term: 3204
    }



### Task2 : Register number of users 

etcd3 api: https://python-etcd3.readthedocs.io/en/latest/usage.html

for all names in table fixedUsers register the appropriate key value pairs


```python
fixedUsers = [
    'Adam',
    'Borys',
    'Cezary',
    'Damian',
    'Emil',
    'Filip',
    'Gustaw',
    'Henryk',
    'Ignacy',
    'Jacek',
    'Kamil',
    'Leon',
    'Marek',
    'Norbert',
    'Oskar',
    'Patryk',
    'Rafał',
    'Stefan',
    'Tadeusz'
]

for u in fixedUsers:
    userKey='{}/{}'.format(cfgRoot, u)
    hostname = socket.gethostname()
    etcd.put(userKey,hostname)
    
```

### Task3: List all users

etcd3 api: https://python-etcd3.readthedocs.io/en/latest/usage.html

List all registered user (tip: use common prefix)


```python
for val in etcd.get_prefix(cfgRoot):
    print(val)
```

    (b'notebook-condafree1py36d4720f40a6d242a0b93099c0882a794a-7bvckh7', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36d4720f40a6d242a0b93099c0882a794a-7bvckh7', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'7', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36933951faff2f46449720793983297309-5f4ln4v', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36fd46088b0ef240789f6dc151c1df28e0-7ckrd6p', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py362d5e52f7d9fc4a24a46795ea3783b0c6-6dsht55', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'temporary', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'Tadeusz', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'condition failed', <etcd3.client.KVMetadata object at 0x7f6f5025bef0>)
    (b'2', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'somevalue', <etcd3.client.KVMetadata object at 0x7f6f5025b470>)
    (b'Adam', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'Borys', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Kamil', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'Leon', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Marek', <etcd3.client.KVMetadata object at 0x7f6f5025bef0>)
    (b'Norbert', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Oskar', <etcd3.client.KVMetadata object at 0x7f6f5025bef0>)
    (b'Patryk', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Rafa\xc5\x82', <etcd3.client.KVMetadata object at 0x7f6f5025bef0>)
    (b'Stefan', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Tadeusz', <etcd3.client.KVMetadata object at 0x7f6f5025bef0>)
    (b'Cezary', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Damian', <etcd3.client.KVMetadata object at 0x7f6f5025bef0>)
    (b'Emil', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Filip', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'Gustaw', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Henryk', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'Ignacy', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Jacek', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-toRefresh', <etcd3.client.KVMetadata object at 0x7f6f5025bdd8>)
    (b'notebook-condafree1py36f0d86649c9624b59a61d942f97342b42-64fspjq', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)


### Task 4 : Same as Task2, but use transaction

etcd3 api: https://python-etcd3.readthedocs.io/en/latest/usage.html

for all names in table fixedUsers register the appropriate key value pairs, use transaction to make it a single request  
(Have you noticed any difference in execution time?)


```python
etcd.transaction(
        compare=[etcd.transactions.version(cfgRoot) == 0],
        success=[etcd.transactions.put('{}/{}'.format(cfgRoot, u), 'user-{}'.format(u)) for u in fixedUsers],
        failure=[etcd.transactions.put('/tmp/failure', 'condtion failed')]
    )
```




    (False, [response_put {
        header {
          revision: 315592
        }
      }])



### Task 5 : Get single key (e.g. status of transaction)

etcd3 api: https://python-etcd3.readthedocs.io/en/latest/usage.html

Check the key you are modifying in on-failure handler in previous task


```python
for i in etcd.get_prefix('/tmp/failure'):
    print(i)
```

    (b'condtion failed', <etcd3.client.KVMetadata object at 0x7f6f5025bbe0>)


### Task 6 : Get range of Keys (Emil -> Oskar) 

etcd3 api: https://python-etcd3.readthedocs.io/en/latest/usage.html

- Get range of keys
- Is it inclusive / exclusive?
- Sort the resposne descending
- Sort the resposne descending by value not by key


```python
r_from = '{}/{}'.format(cfgRoot, 'Emil')
r_to = '{}/{}'.format(cfgRoot, 'Oskar')
for key in etcd.get_range(r_from, r_to):
    print(key)
```

    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274898>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274b00>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274898>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274b00>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274898>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274b00>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274898>)
    (b'7', <etcd3.client.KVMetadata object at 0x7f6f50274b00>)
    (b'notebook-condafree1py36933951faff2f46449720793983297309-5f4ln4v', <etcd3.client.KVMetadata object at 0x7f6f50274898>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274b00>)
    (b'notebook-condafree1py36fd46088b0ef240789f6dc151c1df28e0-7ckrd6p', <etcd3.client.KVMetadata object at 0x7f6f50274898>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274b00>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f50274898>)


### Task 7: Atomic Replace

etcd3 api: https://python-etcd3.readthedocs.io/en/latest/usage.html

Do it a few times, check if value has been replaced depending on condition


```python
etcd.transaction(
        compare=[],
        success=[etcd.transactions.put('{}/{}'.format(cfgRoot, user), 'user-2-{}'.format(user)) for user in fixedUsers],
        failure=[]
    )

for user in fixedUsers:
    result = etcd.replace('{}/{}'.format(cfgRoot, user), 'user-2-{}'.format(user), 'user-3-{}'.format(user))
    print(f'{user}: {result}')

for user in fixedUsers:
    result = etcd.replace('{}/{}'.format(cfgRoot, user), 'user-2-{}'.format(user), 'user-3-{}'.format(user))
    print(f'{user}: {result}')
```

    Adam: True
    Borys: True
    Cezary: True
    Damian: True
    Emil: True
    Filip: True
    Gustaw: True
    Henryk: True
    Ignacy: True
    Jacek: True
    Kamil: True
    Leon: True
    Marek: True
    Norbert: True
    Oskar: True
    Patryk: True
    Rafał: True
    Stefan: True
    Tadeusz: True
    Adam: False
    Borys: False
    Cezary: False
    Damian: False
    Emil: False
    Filip: False
    Gustaw: False
    Henryk: False
    Ignacy: False
    Jacek: False
    Kamil: False
    Leon: False
    Marek: False
    Norbert: False
    Oskar: False
    Patryk: False
    Rafał: False
    Stefan: False
    Tadeusz: False


### Task 8 : Create lease - use it to create expiring key

etcd3 api: https://python-etcd3.readthedocs.io/en/latest/usage.html

You can create a key that will be for limited time
add user that will expire after a few seconds

Tip: Use lease



```python
import time

key='{}/{}'.format(cfgRoot, '10secUser')
val='user-{}'.format('10secUser')

lease = etcd.lease(ttl=5)
etcd.put(key, val, lease=lease)

for key in etcd.get_prefix(cfgRoot):
    print(key)

print("========================================")
time.sleep(6)


for key in etcd.get_prefix(cfgRoot):
    print(key)
```

    (b'notebook-condafree1py36d4720f40a6d242a0b93099c0882a794a-7bvckh7', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-10secUser', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'notebook-condafree1py36d4720f40a6d242a0b93099c0882a794a-7bvckh7', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Adam', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'user-3-Borys', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Cezary', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'user-3-Damian', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Emil', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'user-3-Filip', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Gustaw', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'user-3-Henryk', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Ignacy', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'user-3-Jacek', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Kamil', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'7', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'notebook-condafree1py36933951faff2f46449720793983297309-5f4ln4v', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'user-3-Leon', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'notebook-condafree1py36fd46088b0ef240789f6dc151c1df28e0-7ckrd6p', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'user-3-Marek', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Norbert', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'user-3-Oskar', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Patryk', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'notebook-condafree1py362d5e52f7d9fc4a24a46795ea3783b0c6-6dsht55', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Rafa\xc5\x82', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'user-3-Stefan', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Tadeusz', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'temporary', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Tadeusz', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'condition failed', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'2', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'somevalue', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Adam', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'Borys', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Kamil', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'Leon', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Marek', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'Norbert', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Oskar', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'Patryk', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Rafa\xc5\x82', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'Stefan', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Tadeusz', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'Cezary', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Damian', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'Emil', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Filip', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'Gustaw', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Henryk', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'Ignacy', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Jacek', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-toRefresh', <etcd3.client.KVMetadata object at 0x7f6f502848d0>)
    (b'notebook-condafree1py36f0d86649c9624b59a61d942f97342b42-64fspjq', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    ========================================
    (b'notebook-condafree1py36d4720f40a6d242a0b93099c0882a794a-7bvckh7', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'notebook-condafree1py36d4720f40a6d242a0b93099c0882a794a-7bvckh7', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Adam', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'user-3-Borys', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Cezary', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'user-3-Damian', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Emil', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'user-3-Filip', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Gustaw', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'user-3-Henryk', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Ignacy', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'user-3-Jacek', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Kamil', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'7', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'notebook-condafree1py36933951faff2f46449720793983297309-5f4ln4v', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'user-3-Leon', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'notebook-condafree1py36fd46088b0ef240789f6dc151c1df28e0-7ckrd6p', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'user-3-Marek', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Norbert', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'user-3-Oskar', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Patryk', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'notebook-condafree1py362d5e52f7d9fc4a24a46795ea3783b0c6-6dsht55', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Rafa\xc5\x82', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'user-3-Stefan', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-3-Tadeusz', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'temporary', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Tadeusz', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'condition failed', <etcd3.client.KVMetadata object at 0x7f6f50284780>)
    (b'2', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'somevalue', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Adam', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'Borys', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Kamil', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'Leon', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Marek', <etcd3.client.KVMetadata object at 0x7f6f50284780>)
    (b'Norbert', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Oskar', <etcd3.client.KVMetadata object at 0x7f6f50284780>)
    (b'Patryk', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Rafa\xc5\x82', <etcd3.client.KVMetadata object at 0x7f6f50284780>)
    (b'Stefan', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Tadeusz', <etcd3.client.KVMetadata object at 0x7f6f50284780>)
    (b'Cezary', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Damian', <etcd3.client.KVMetadata object at 0x7f6f50284780>)
    (b'Emil', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Filip', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'Gustaw', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Henryk', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'Ignacy', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'Jacek', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'notebook-condafree1py36e38f78dbb72f42428025b7fc3d0c5819-6fwqqhq', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-toRefresh', <etcd3.client.KVMetadata object at 0x7f6f50284978>)
    (b'notebook-condafree1py36f0d86649c9624b59a61d942f97342b42-64fspjq', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)


### Task 9 : Create key that will expire after you close the connection to etcd

Tip: use threading library to refresh your lease


```python
import threading

Lease = etcd.lease(ttl=10)

def refresh_lease():
    while(True):
        Lease.refresh()
        time.sleep(1)

key='{}/{}'.format(cfgRoot, 'toRefresh')
val='user-{}'.format('toRefresh')


etcd.put(key, val, lease=Lease)

t = threading.Thread(target=refresh_lease)
t.start()

print(etcd.get(key))
time.sleep(15)
print(etcd.get(key))
```

    (b'user-toRefresh', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)
    (b'user-toRefresh', <etcd3.client.KVMetadata object at 0x7f6f5025be48>)


### Task 9: Use lock to protect section of code

etcd3 api: https://python-etcd3.readthedocs.io/en/latest/usage.html


```python
import time
# it was tested with freiends using the same lock
with etcd.lock('lock-1', ttl=20) as lock:
    print('a')
    print(f'Is acquaired? {lock.is_acquired()}')
    lock.acquire()
    print('b')
    time.sleep(3)
    print('c')
    lock.release()

```

    a
    Is acquaired? True
    b
    c


### Task 10 Watch key

etcd3 api: https://python-etcd3.readthedocs.io/en/latest/usage.html

This cell will lock this notebook on waiting  
After running it create a new notebook and try to add new user


```python
# tested with friends, one puts value in db : etcd.put('/lease/watch/friends', 'xd-2')
def etcd_call(cb):
    print(cb)

etcd.add_watch_callback(key='/lease/watch/friends', callback=etcd_call)
```




    1



    <etcd3.watch.WatchResponse object at 0x7f6f5025b3c8>
    <etcd3.watch.WatchResponse object at 0x7f6f5025b5c0>



```python

# etcd.put('/lease/watch/friends', 'xd-2')
```
