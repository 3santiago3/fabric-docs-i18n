私有数据
============
Private Data
============

.. note:: 本主题假设你已经理解了在 `私有数据文档 <private-data/private-data.html>`_ 中所描述概念。

.. note:: This topic assumes an understanding of the conceptual material in the
          `documentation on private data <private-data/private-data.html>`_.

私有数据集合定义
----------------------------------

Private data collection definition
----------------------------------

一个集合定义包含了一个或者多个集合，每个集合具有一个策略列出了在集合中的所有组织，还包括用来控制在背书阶段私有数据的传播所使用的属性，另外还有一个可选项来决定数据是否会被删除。
Beginning with the Fabric chaincode lifecycle introduced with Fabric v2.0, the
collection definition is part of the chaincode definition. The collection is
approved by channel members, and then deployed when the chaincode definition
is committed to the channel. The collection file needs to be the same for all
channel members. If you are using the peer CLI to approve and commit the
chaincode definition, use the ``--collections-config`` flag to specify the path
to the collection definition file. If you are using the Fabric SDK for Node.js,
visit `How to install and start your chaincode <https://hyperledger.github.io/fabric-sdk-node/master/tutorial-chaincode-lifecycle.html>`_.
To use the `previous lifecycle process <https://hyperledger-fabric.readthedocs.io/en/release-1.4/chaincode4noah.html>`_ to deploy a private data collection,
use the ``--collections-config`` flag when `instantiating your chaincode <https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html#peer-chaincode-instantiate>`_.

A collection definition contains one or more collections, each having a policy
definition listing the organizations in the collection, as well as properties
used to control dissemination of private data at endorsement time and,
optionally, whether the data will be purged.

集合定义由下边的属性组成：

Beginning with the Fabric chaincode lifecycle introduced with Fabric v2.0, the
collection definition is part of the chaincode definition. The collection is
approved by channel members, and then deployed when the chaincode definition
is committed to the channel. The collection file needs to be the same for all
channel members. If you are using the peer CLI to approve and commit the
chaincode definition, use the ``--collections-config`` flag to specify the path
to the collection definition file. If you are using the Fabric SDK for Node.js,
visit `How to install and start your chaincode <https://hyperledger.github.io/fabric-sdk-node/{BRANCH}/tutorial-chaincode-lifecycle.html>`_.
To use the `previous lifecycle process <https://hyperledger-fabric.readthedocs.io/en/release-1.4/chaincode4noah.html>`_ to deploy a private data collection,
use the ``--collections-config`` flag when `instantiating your chaincode <https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html#peer-chaincode-instantiate>`_.

集合定义由下边的属性组成：

Collection definitions are composed of the following properties:

* ``name``: 集合的名字.

* ``name``: Name of the collection.

* ``policy``: 私有数据集合分发策略，它通过 ``Signature`` 策略语法定义了允许哪些组织的 Peer 节点持久化集合数据。为了支持读/写交易，私有数据的分发策略必须要定义一个比链码背书策略更大范围的一个组织的集合，因为 Peer 节点必须要拥有这些私有数据才能来对这些交易提案进行背书。比如，在一个具有10个组织的通道中，其中5个组织可能会被包含在一个私有数据集合的分发策略中，但是背书策略可能会调用其中任意的3个组织来进行背书。

* ``policy``: The private data collection distribution policy defines which
  organizations' peers are allowed to persist the collection data expressed using
  the ``Signature`` policy syntax, with each member being included in an ``OR``
  signature policy list. To support read/write transactions, the private data
  distribution policy must define a broader set of organizations than the chaincode
  endorsement policy, as peers must have the private data in order to endorse
  proposed transactions. For example, in a channel with ten organizations,
  five of the organizations might be included in a private data collection
  distribution policy, but the endorsement policy might call for any three
  of the organizations to endorse.

* ``requiredPeerCount``: 在节点为背书签名并将提案响应返回给客户端前，每个背书节点必须将私有数据分发到的节点(在被授权的组织当中)的最小数量。将传播作为背书的一个条件可以确保即使背书背书不可用，私有数据在网络中也还是可用的。``requiredPeerCount`` 设为 ``0`` 代表分发并不是 **必须** 的，但是如果 ``maxPeerCount`` 比0大的话，就需要分发。通常不建议 ``requiredPeerCount`` 设为 ``0`` 通常，因为那会造成在背书节点不可用的时候，网络中的私有数据可能会丢失。通常在背书的时候你会希望分发私有数据到多个节点以保证网络中私有数据的冗余存储。

* ``requiredPeerCount``: Minimum number of peers (across authorized organizations)
  that each endorsing peer must successfully disseminate private data to before the
  peer signs the endorsement and returns the proposal response back to the client.
  Requiring dissemination as a condition of endorsement will ensure that private data
  is available in the network even if the endorsing peer(s) become unavailable. When
  ``requiredPeerCount`` is ``0``, it means that no distribution is **required**,
  but there may be some distribution if ``maxPeerCount`` is greater than zero. A
  ``requiredPeerCount`` of ``0`` would typically not be recommended, as it could
  lead to loss of private data in the network if the endorsing peer(s) becomes unavailable.
  Typically you would want to require at least some distribution of the private
  data at endorsement time to ensure redundancy of the private data on multiple
  peers in the network.

* ``maxPeerCount``: 为了数据的冗余存储，每个背书节点将会尝试将私有数据分发到的其他节点（在被授权的组织中) 的最大数量。如果在背书和提交之间一个背书节点不可用了，其他节点就可以在背书的时候从已经收到私有数据的节点拉取私有数据。如果这个值被设置为 ``0``，私有数据在背书的时候就不会被分发，这会在提交的时候强制节点从授权的背书节点上拉取私有数据。

* ``maxPeerCount``: For data redundancy purposes, the maximum number of other
  peers (across authorized organizations) that each endorsing peer will attempt
  to distribute the private data to. If an endorsing peer becomes unavailable between
  endorsement time and commit time, other peers that are collection members but who
  did not yet receive the private data at endorsement time, will be able to pull
  the private data from peers the private data was disseminated to. If this value
  is set to ``0``, the private data is not disseminated at endorsement time,
  forcing private data pulls against endorsing peers on all authorized peers at
  commit time.

* ``blockToLive``: 代表了数据应该以区块的形式在私有数据库中存在多久。数据会在私有数据库上存在指定数量个区块，然后它会被删除，然后链码就无法查询到该数据，其他节点也无法请求到该数据。如果要无限期的保持私有数据，也就是从来不删除私有数据的话，将 ``blockToLive`` 设置为 ``0``。

* ``blockToLive``: Represents how long the data should live on the private
  database in terms of blocks. The data will live for this specified number of
  blocks on the private database and after that it will get purged, making this
  data obsolete from the network so that it cannot be queried from chaincode,
  and cannot be made available to requesting peers. To keep private data
  indefinitely, that is, to never purge private data, set the ``blockToLive``
  property to ``0``.

* ``memberOnlyRead``: ``true`` 值表示节点自动会强制只有属于这些集合的组织的客户端才可以读取私有数据。如果一个非成员组织的客户端试图执行一个链码方法来读取私有数据的话，会结束链码的调用并产生错误。如果你想在单独的链码方法中进行更细粒度的访问控制的话，可以使用 ``false`` 值。

* ``memberOnlyRead``: a value of ``true`` indicates that peers automatically
  enforce that only clients belonging to one of the collection member organizations
  are allowed read access to private data. If a client from a non-member org
  attempts to execute a chaincode function that performs a read of a private data key,
  the chaincode invocation is terminated with an error. Utilize a value of
  ``false`` if you would like to encode more granular access control within
  individual chaincode functions.

* ``memberOnlyWrite``: a value of ``true`` indicates that peers automatically
  enforce that only clients belonging to one of the collection member organizations
  are allowed to write private data from chaincode. If a client from a non-member org
  attempts to execute a chaincode function that performs a write on a private data key,
  the chaincode invocation is terminated with an error. Utilize a value of
  ``false`` if you would like to encode more granular access control within
  individual chaincode functions, for example you may want certain clients
  from non-member organization to be able to create private data in a certain
  collection.

* ``endorsementPolicy``: An optional endorsement policy to utilize for the
  collection that overrides the chaincode level endorsement policy. A
  collection level endorsement policy may be specified in the form of a
  ``signaturePolicy`` or may be a ``channelConfigPolicy`` reference to
  an existing policy from the channel configuration. The ``endorsementPolicy``
  may be the same as the collection distribution ``policy``, or may require
  fewer or additional organization peers.

下边是一个集合定义的 JSON 文件示例，一个包含了两个集合定义的数组：

Here is a sample collection definition JSON file, containing an array of two
collection definitions:

.. code:: bash

 [
  {
     "name": "collectionMarbles",
     "policy": "OR('Org1MSP.member', 'Org2MSP.member')",
     "requiredPeerCount": 0,
     "maxPeerCount": 3,
     "blockToLive":1000000,
     "memberOnlyRead": true,
     "memberOnlyWrite": true
  },
  {
     "name": "collectionMarblePrivateDetails",
     "policy": "OR('Org1MSP.member')",
     "requiredPeerCount": 0,
     "maxPeerCount": 3,
     "blockToLive":3,
     "memberOnlyRead": true,
     "memberOnlyWrite":true,
     "endorsementPolicy": {
       "signaturePolicy": "OR('Org1MSP.member')"
     }
  }
 ]

This example uses the organizations from the Fabric test network, ``Org1`` and
``Org2`` . The policy in the  ``collectionMarbles`` definition authorizes both
organizations to the private data. This is a typical configuration when the
chaincode data needs to remain private from the ordering service nodes. However,
the policy in the ``collectionMarblePrivateDetails`` definition restricts access
to a subset of organizations in the channel (in this case ``Org1`` ). Additionally,
writing to this collection requires endorsement from a ``Org1`` peer, even
though the chaincode level endorsement policy may require endorsement from
``Org1`` or ``Org2``. And since "memberOnlyWrite" is true, only clients from
``Org1`` may invoke chaincode that writes to the private data collection.
In this way you can control which organizations are entrusted to write to certain
private data collections.

This example uses the organizations from the Fabric test network, ``Org1`` and
``Org2``. The policy in the  ``collectionMarbles`` definition authorizes both
organizations to the private data. This is a typical configuration when the
chaincode data needs to remain private from the ordering service nodes. However,
the policy in the ``collectionMarblePrivateDetails`` definition restricts access
to a subset of organizations in the channel (in this case ``Org1`` ). Additionally,
writing to this collection requires endorsement from an ``Org1`` peer, even
though the chaincode level endorsement policy may require endorsement from
``Org1`` or ``Org2``. And since "memberOnlyWrite" is true, only clients from
``Org1`` may invoke chaincode that writes to the private data collection.
In this way you can control which organizations are entrusted to write to certain
private data collections.

私有数据分发
-----------------------------------

Private data dissemination
--------------------------

由于私有数据不会被包含在提交到排序服务的交易中，因此也就不会被包含在区块中，背书节点扮演着将私有数据分发给其他授权组织的节点的重要角色。这确保了私有数据在背书节点完成背书之后变成不可用的时候的可用性。为了辅助分发，在集合定义中的 ``maxPeerCount`` 和 ``requiredPeerCount`` 属性控制了在背书的时候分发的数量。

Since private data is not included in the transactions that get submitted to
the ordering service, and therefore not included in the blocks that get distributed
to all peers in a channel, the endorsing peer plays an important role in
disseminating private data to other peers of authorized organizations. This ensures
the availability of private data in the channel's collection, even if endorsing
peers become unavailable after their endorsement. To assist with this dissemination,
the  ``maxPeerCount`` and ``requiredPeerCount`` properties in the collection definition
control the degree of dissemination at endorsement time.

如果背书节点不能够成功地将私有数据分发到至少 ``requiredPeerCount`` 的要求，它将会返回一个错误给客户端。背书节点会尝试将私有数据分发到不同组织的节点，来确保每个被授权的组织具有私有数据的一个副本。因为交易在链码执行期间还没有被提交，背书节点和接收节点除了在它们的区块链之外，还在一个本地的 ``临时存储（transient store）`` 中存储了一个私有数据副本，直到交易被提交。

If the endorsing peer cannot successfully disseminate the private data to at least
the ``requiredPeerCount``, it will return an error back to the client. The endorsing
peer will attempt to disseminate the private data to peers of different organizations,
in an effort to ensure that each authorized organization has a copy of the private
data. Since transactions are not committed at chaincode execution time, the endorsing
peer and recipient peers store a copy of the private data in a local ``transient store``
alongside their blockchain until the transaction is committed.

当一个被授权的节点在提交的时候，如果他们的临时存储中没有私有数据的副本（或者是因为他们不是一个背书节点，或者是因为他们在背书的时候没有接收到私有数据），他们会尝试从其他的被授权的节点那里拉取私有数据，尝试会*持续一个可配置的时间长度* ，在时间可以通过节点配置文件 ``core.yaml`` 中的属性 ``peer.gossip.pvtData.pullRetryThreshold`` 进行配置。

When authorized peers do not have a copy of the private data in their transient
data store at commit time (either because they were not an endorsing peer or because
they did not receive the private data via dissemination at endorsement time),
they will attempt to pull the private data from another authorized
peer, *for a configurable amount of time* based on the peer property
``peer.gossip.pvtData.pullRetryThreshold`` in the peer configuration ``core.yaml``
file.

.. note::
      只有当提出请求的节点是私有数据分发策略定义的集合中的一员的时候，被询问的节点才会返回私有数据。

.. note:: The peers being asked for private data will only return the private data
          if the requesting peer is a member of the collection as defined by the
          private data dissemination policy.

当使用 ``pullRetryThreshold`` 时候需要考虑的问题：

Considerations when using ``pullRetryThreshold``:

* 如果提出请求的节点能够在 ``pullRetryThreshold`` 时间内拿到私有数据的话，它将会把交易提交到自己的账本（包括私有数据的哈希值），并且将私有数据存储在与其他的通道状态数据进行了逻辑隔离的状态数据库中。

* If the requesting peer is able to retrieve the private data within the
  ``pullRetryThreshold``, it will commit the transaction to its ledger
  (including the private data hash), and store the private data in its
  state database, logically separated from other channel state data.

* 如果提出请求的节点没能在 ``pullRetryThreshold`` 时间内拿到私有数据的话，它将会把交易提交到自己的账本（包括私有数据的哈希值），但是不会存储私有数据。

* If the requesting peer is not able to retrieve the private data within
  the ``pullRetryThreshold``, it will commit the transaction to it’s blockchain
  (including the private data hash), without the private data.

* 如果某个节点有资格拥有私有数据，却没有得到的话，这个节点就无法为将来会引用这个丢失的私有数据的交易进行背书，背书时会发现无法查询到键 （基于在状态数据库中主键的哈希值），并且链码将会收到一个错误。

* If the peer was entitled to the private data but it is missing, then
  that peer will not be able to endorse future transactions that reference
  the missing private data - a chaincode query for a key that is missing will
  be detected (based on the presence of the key’s hash in the state database),
  and the chaincode will receive an error.

因此，将 ``requiredPeerCount`` 和 ``maxPeerCount`` 设置成足够大的值来确保在你的通道中的私有数据的可用性是非常重要的。比如，如果在交易提交之前，每个背书节点都不可用了，``requiredPeerCount`` 和 ``maxPeerCount`` 属性将会确保私有数据在其他的节点上是可用的。

Therefore, it is important to set the ``requiredPeerCount`` and ``maxPeerCount``
properties large enough to ensure the availability of private data in your
channel. For example, if each of the endorsing peers become unavailable
before the transaction commits, the ``requiredPeerCount`` and ``maxPeerCount``
properties will have ensured the private data is available on other peers.

.. note::
      为了让集合能够工作，正确配置跨组织的 gossip 非常重要的。请阅读 :doc:`gossip`，尤其注意“锚节点”这部分。

.. note:: For collections to work, it is important to have cross organizational
          gossip configured correctly. Refer to our documentation on :doc:`gossip`,
          paying particular attention to the "anchor peers" and "external endpoint"
          configuration.

从链码中引用集合
--------------------------------------

Referencing collections from chaincode
--------------------------------------

我们可以用 `shim APIs <https://godoc.org/github.com/hyperledger/fabric-chaincode-go/shim>`_ 设置和取回私有数据。

A set of `shim APIs <https://godoc.org/github.com/hyperledger/fabric-chaincode-go/shim>`_
are available for setting and retrieving private data.

相同的链码数据操作也可以应用到通道状态数据和私有数据上，但是对于私有数据，要在链码 API 中指定和数据相关的集合的名字，比如 ``PutPrivateData(collection,key,value)`` 和 ``GetPrivateData(collection,key)``。

The same chaincode data operations can be applied to channel state data and
private data, but in the case of private data, a collection name is specified
along with the data in the chaincode APIs, for example
``PutPrivateData(collection,key,value)`` and ``GetPrivateData(collection,key)``.

一个链码可以引用多个集合。

A single chaincode can reference multiple collections.

Referencing implicit collections from chaincode
-----------------------------------------------

Starting in v2.0, an implicit private data collection can be used for each
organization in a channel, so that you don't have to define collections if you'd
like to utilize per-organization collections. Each org-specific implicit collection
has a distribution policy and endorsement policy of the matching organization.
You can therefore utilize implicit collections for use cases where you'd like
to ensure that a specific organization has written to a collection key namespace.
The v2.0 chaincode lifecycle uses implicit collections to track which organizations
have approved a chaincode definition. Similarly, you can use implicit collections
in application chaincode to track which organizations have approved or voted
for some change in state.

To write and read an implicit private data collection key, in the ``PutPrivateData``
and ``GetPrivateData`` chaincode APIs, specify the collection parameter as
``"_implicit_org_<MSPID>"``, for example ``"_implicit_org_Org1MSP"``.

.. note:: Application defined collection names are not allowed to start with an underscore,
          therefore there is no chance for an implicit collection name to collide
          with an application defined collection name.

How to pass private data in a chaincode proposal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

因为链码提案被存储在区块链上，不要把私有数据包含在链码提案中也是非常重要的。在链码提案中有一个特殊的字段 ``transient``，可以用它把私有数据来从客户端（或者链码将用来生成私有数据的数据）传递给节点上的链码调用。链码可以通过调用 `GetTransient() API <https://godoc.org/github.com/hyperledger/fabric-chaincode-go/shim#ChaincodeStub.GetTransient>`_ 来获取 ``transient`` 字段。这个 ``transient`` 字段会从通道交易中被排除。

Since the chaincode proposal gets stored on the blockchain, it is also important
not to include private data in the main part of the chaincode proposal. A special
field in the chaincode proposal called the ``transient`` field can be used to pass
private data from the client (or data that chaincode will use to generate private
data), to chaincode invocation on the peer.  The chaincode can retrieve the
``transient`` field by calling the `GetTransient() API <https://godoc.org/github.com/hyperledger/fabric-chaincode-go/shim#ChaincodeStub.GetTransient>`_.
This ``transient`` field gets excluded from the channel transaction.

Protecting private data content
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If the private data is relatively simple and predictable (e.g. transaction dollar
amount), channel members who are not authorized to the private data collection
could try to guess the content of the private data via brute force hashing of
the domain space, in hopes of finding a match with the private data hash on the
chain. Private data that is predictable should therefore include a random "salt"
that is concatenated with the private data key and included in the private data
value, so that a matching hash cannot realistically be found via brute force.
The random "salt" can be generated at the client side (e.g. by sampling a secure
psuedo-random source) and then passed along with the private data in the transient
field at the time of chaincode invocation.

Protecting private data content
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If the private data is relatively simple and predictable (e.g. transaction dollar
amount), channel members who are not authorized to the private data collection
could try to guess the content of the private data via brute force hashing of
the domain space, in hopes of finding a match with the private data hash on the
chain. Private data that is predictable should therefore include a random "salt"
that is concatenated with the private data key and included in the private data
value, so that a matching hash cannot realistically be found via brute force.
The random "salt" can be generated at the client side (e.g. by sampling a secure
pseudo-random source) and then passed along with the private data in the transient
field at the time of chaincode invocation.

私有数据的访问控制
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Access control for private data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Until version 1.3, access control to private data based on collection membership
was enforced for peers only. Access control based on the organization of the
chaincode proposal submitter was required to be encoded in chaincode logic.
Collection configuration options ``memberOnlyRead`` (since version v1.4) and
``memberOnlyWrite`` (since version v2.0) can automatically enforce that the chaincode
proposal submitter must be from a collection member in order to read or write
private data keys. For more information about collection
configuration definitions and how to set them, refer back to the
`Private data collection definition`_  section of this topic.

.. note:: If you would like more granular access control, you can set
          ``memberOnlyRead`` and ``memberOnlyWrite`` to false. You can then apply your
          own access control logic in chaincode, for example by calling the GetCreator()
          chaincode API or using the client identity
          `chaincode library <https://godoc.org/github.com/hyperledger/fabric-chaincode-go/shim#ChaincodeStub.GetCreator>`__ .

查询私有数据
~~~~~~~~~~~~~~~~~~~~~

Querying Private Data
~~~~~~~~~~~~~~~~~~~~~

私有集合数据能够像常见的通道数据那样使用 shim API 来进行查询：

Private data collection can be queried just like normal channel data, using
shim APIs:

* ``GetPrivateDataByRange(collection, startKey, endKey string)``
* ``GetPrivateDataByPartialCompositeKey(collection, objectType string, keys []string)``

对于 CouchDB 状态数据库，可以使用 shim API 查询 JSON 内容：

And for the CouchDB state database, JSON content queries can be passed using the
shim API:

* ``GetPrivateDataQueryResult(collection, query string)``

限制：

Limitations:

* 客户端调用执行范围查询或者富查询链码的时候应该知道，根据上边关于私有数据分发部分的解释，如果他们查询的节点有丢失的私有数据的话，他们可能会接收到结果集的一个子集。客户端可以查询多个节点并且比较返回的结果，以确定一个节点是否丢失了结果集中的部分数据。

* Clients that call chaincode that executes range or rich JSON queries should be aware
  that they may receive a subset of the result set, if the peer they query has missing
  private data, based on the explanation in Private Data Dissemination section
  above.  Clients can query multiple peers and compare the results to
  determine if a peer may be missing some of the result set.
* Chaincode that executes range or rich JSON queries and updates data in a single
  transaction is not supported, as the query results cannot be validated on the peers
  that don’t have access to the private data, or on peers that are missing the
  private data that they have access to. If a chaincode invocation both queries
  and updates private data, the proposal request will return an error. If your application
  can tolerate result set changes between chaincode execution and validation/commit time,
  then you could call one chaincode function to perform the query, and then call a second
  chaincode function to make the updates. Note that calls to GetPrivateData() to retrieve
  individual keys can be made in the same transaction as PutPrivateData() calls, since
  all peers can validate key reads based on the hashed key version.

* 不支持在单个交易中既执行范围查询或者富查询并且更新数据，因为查询结果无法在以下类型的节点上进行验证：不能访问私有数据的节点或者对于那些他们可以访问相关的私有数据但是私有数据是丢失的。如果一个链码的调用既查询又更新私有数据的话，这个提案请求将会返回一个错误。如果你的应用程序能够容忍在链码执行和验证/提交阶段结果集的变动，那么你可以调用一个链码方法来执行这个查询，然后再调用第二个链码方法来执行变更。注意，调用 GetPrivateData() 来获取单独的键值可以跟 PutPrivateData() 调用放在同一个交易中，因为所有的节点都能够基于键版本的哈希来验证键的读取。

Using Indexes with collections
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在集合中使用索引
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The topic :doc:`couchdb_as_state_database` describes indexes that can be
applied to the channel’s state database to enable JSON content queries, by
packaging indexes in a ``META-INF/statedb/couchdb/indexes`` directory at chaincode
installation time.  Similarly, indexes can also be applied to private data
collections, by packaging indexes in a ``META-INF/statedb/couchdb/collections/<collection_name>/indexes``
directory. An example index is available `here <https://github.com/hyperledger/fabric-samples/blob/{BRANCH}/chaincode/marbles02_private/go/META-INF/statedb/couchdb/collections/collectionMarbles/indexes/indexOwner.json>`_.

:doc:`couchdb_as_state_database` 章节讲解了可以在安装阶段，通过将索引打包在一个 ``META-INF/statedb/couchdb/indexes`` 的路径下的方式，将索引应用到通道的状态数据库。类似的，也可以通过将索引打包在一个 ``META-INF/statedb/couchdb/collections/<collection_name>/indexes`` 路径下的方式将索引应用到私有数据集合中。一个索引的实例可以查看 `这里 <https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02_private/go/META-INF/statedb/couchdb/collections/collectionMarbles/indexes/indexOwner.json>`_。

Considerations when using private data
--------------------------------------

使用私有数据时的思考
--------------------------------------

Private data purging
~~~~~~~~~~~~~~~~~~~~

私有数据的删除
~~~~~~~~~~~~~~~~~~~~

Private data can be periodically purged from peers. For more details,
see the ``blockToLive`` collection definition property above.

Peer 可以周期性地删除私有数据。更多细节请查看上边集合定义属性中的 ``blockToLive`` 。

Additionally, recall that prior to commit, peers store private data in a local
transient data store. This data automatically gets purged when the transaction
commits.  But if a transaction was never submitted to the channel and
therefore never committed, the private data would remain in each peer’s
transient store.  This data is purged from the transient store after a
configurable number blocks by using the peer’s
``peer.gossip.pvtData.transientstoreMaxBlockRetention`` property in the peer
``core.yaml`` file.

另外，重申一下，在提交之前，私有数据存储在 Peer 节点的本地临时数据存储中。这些数据在交易提交之后会自动被删除。但是如果交易没有被提交，私有数据就会一直保存在临时数据存储中。Peer 节点会根据配置文件 ``core.yaml`` 中的 ``peer.gossip.pvtData.transientstoreMaxBlockRetention`` 的配置周期性的删除临时存储中的数据。

Updating a collection definition
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Updating a collection definition
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To update a collection definition or add a new collection, you can update
the chaincode definition and pass the new collection configuration
in the chaincode approve and commit transactions, for example using the ``--collections-config``
flag if using the CLI. If a collection configuration is specified when updating
the chaincode definition, a definition for each of the existing collections must be
included.

To update a collection definition or add a new collection, you can update
the chaincode definition and pass the new collection configuration
in the chaincode approve and commit transactions, for example using the ``--collections-config``
flag if using the CLI. If a collection configuration is specified when updating
the chaincode definition, a definition for each of the existing collections must be
included.

When updating a chaincode definition, you can add new private data collections,
and update existing private data collections, for example to add new
members to an existing collection or change one of the collection definition
properties. Note that you cannot update the collection name or the
blockToLive property, since a consistent blockToLive is required
regardless of a peer's block height.

When updating a chaincode definition, you can add new private data collections,
and update existing private data collections, for example to add new
members to an existing collection or change one of the collection definition
properties. Note that you cannot update the collection name or the
blockToLive property, since a consistent blockToLive is required
regardless of a peer's block height.

Collection updates becomes effective when a peer commits the block with the updated
chaincode definition. Note that collections cannot be
deleted, as there may be prior private data hashes on the channel’s blockchain
that cannot be removed.

Collection updates becomes effective when a peer commits the block with the updated
chaincode definition. Note that collections cannot be
deleted, as there may be prior private data hashes on the channel’s blockchain
that cannot be removed.

Private data reconciliation
~~~~~~~~~~~~~~~~~~~~~~~~~~~

私有数据对账
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting in v1.4, peers of organizations that are added to an existing collection
will automatically fetch private data that was committed to the collection before
they joined the collection.

从 v1.4 开始，加入到已存在的集合中的 Peer 节点在私有数据加入到集合之前，可以自动获取提交到集合的私有数据。

This private data "reconciliation" also applies to peers that
were entitled to receive private data but did not yet receive it --- because of
a network failure, for example --- by keeping track of private data that was "missing"
at the time of block commit.

私有数据“对账”也应用在 Peer 节点上，用于确认该接收却未接收到的私有数据，比如由于网络原因没有收到的。以此来追踪在区块提交期间“丢失”的私有数据。

Private data reconciliation occurs periodically based on the
``peer.gossip.pvtData.reconciliationEnabled`` and ``peer.gossip.pvtData.reconcileSleepInterval``
properties in core.yaml. The peer will periodically attempt to fetch the private
data from other collection member peers that are expected to have it.

私有数据对账根据 core.yaml 文件中的属性 ``peer.gossip.pvtData.reconciliationEnabled`` 和 ``peer.gossip.pvtData.reconcileSleepInterval`` 周期性的发生。Peer 节点会从集合成员节点中定期获取私有数据。

Note that this private data reconciliation feature only works on peers running
v1.4 or later of Fabric.

注意私有数据对账特性只适用于 v1.4 以上的 Fabric 节点。
