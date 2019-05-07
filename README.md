# testtask for CASP
## Task 1 & 2:

RUN: ``` docker-compose up -d build ```

CHECK: 
``` curl http://localhost:8181;
curl http://localhost;
curl --insecure https://localhost 
```

## Task 3:

RUN: ``` ansible-playbook playbook-testtask.yml ```

Check: the same as for tasks 1 & 2

## Task 4:

RUN: ``` docker-compose up -d build ```
Copy or edit your host file from host at task4 folder

CHECK:   http://www.test.domain:8182/ .
