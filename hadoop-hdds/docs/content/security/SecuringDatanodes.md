---
title: "Securing Datanodes"
date: "2019-04-03"
weight: 3
menu:
   main:
      parent: Security
summary:  Explains different modes of securing data nodes. These range from kerberos to auto approval.
icon: th
---
<!---
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->


Datanodes under Hadoop is traditionally secured by creating a Keytab file on
the datanodes. With Ozone, we have moved away to using datanode
certificates. That is, Kerberos on datanodes is not needed in case of a
secure Ozone cluster.

However, we support the legacy Kerberos based Authentication to make it easy
for the current set of users.The HDFS configuration keys are the following
that is setup in  hdfs-site.xml.

Property|Description
--------|--------------
dfs.datanode.kerberos.principal|The datanode service principal. <br/> e.g. dn/_HOST@REALM.COM
dfs.datanode.kerberos.keytab.file| The keytab file used by datanode daemon to login as its service principal.
hdds.datanode.http.auth.kerberos.principal| Datanode http server service principal.
hdds.datanode.http.auth.kerberos.keytab| The keytab file used by datanode http server to login as its service principal.


## How a datanode becomes secure.

Under Ozone, when a datanode boots up and discovers SCM's address, the first
thing that datanode does is to create a private key and send a certificate
request to the SCM.

<h3>Certificate Approval via Kerberos <span class="badge badge-secondary">Current Model</span></h3>
SCM has a built-in CA, and SCM has to approve this request. If the datanode
already has a Kerberos key tab, then SCM will trust Kerberos credentials and
issue a certificate automatically.


<h3>Manual Approval <span class="badge badge-primary">In Progress</span></h3>
If these are brand new datanodes and Kerberos key tabs are not present at the
datanodes, then this request for the datanodes identity certificate is
queued up for approval from the administrator(This is work in progress,
not committed in Ozone yet). In other words, the chain of trust is established
by the administrator of the cluster.

<h3>Automatic Approval <span class="badge badge-secondary">In Progress</span></h3>
If you running under an container orchestrator like  Kubernetes, we rely on
Kubernetes to create a one-time token that will be given to datanode during
boot time to prove the identity of the datanode container (This is also work
in progress.)


Once a certificate is issued, a datanode is secure and Ozone manager can
issue block tokens. If there is no datanode certificates or the SCM's root
certificate is not present in the datanode, then datanode will register
itself and download the SCM's root certificate as well get the certificates
for itself.
