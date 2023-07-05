> /*
> mongo yandex cloud
> 1GB RAM 20% ssd 5Gb
> */

#### Task list

- [x] Create db
- [x] mongoimport csv (titanic db)
- [x] check rows
- [x] update rows
- [x] create index
- [x] check index
- [x] drop db

## Start

wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

apt update

apt-get install -y mongodb-org

sudo systemctl start mongod
sudo systemctl status mongod
--Active: active (running)

mongosh
--version mongodb 6.0
--test

show dbs
--admin   40.00 KiB
--config  12.00 KiB
--local   40.00 KiB

### import db titanic people

mongoimport -d titanic -c person --type csv --file /home/nikko/titanic.csv --headerline
--connected to: mongodb://localhost/
--887 document(s) imported successfully. 0 document(s) failed to import.

### Check DB

show dbs
--titanic  88.00 KiB

--set db
use titanic

--check count rows
titanic> db.person.find().count()
887

--show limit 1
titanic> db.person.find().limit(1)
[
{
_id: ObjectId("64a5ca977c1df983a3ca7491"),
Survived: 0,
Pclass: 3,
Name: 'Mr. Owen Harris Braund',
Sex: 'male',
Age: 22,
'Siblings/Spouses Aboard': 1,
'Parents/Children Aboard': 0,
Fare: 7.25
}
]

--UPDATE ALL ROWS. ADD NEW COLUMN
db.person.updateMany({},{$set:{"LoadYear":2023}}) ;
{
acknowledged: true,
insertedId: null,
matchedCount: 887,
modifiedCount: 887,
upsertedCount: 0
}

--Get MALE count
titanic> db.person.find({ Sex:'male' }).count()
573

--Get NOT MALE count
titanic> db.person.find({ Sex:{ $ne: 'male'} }).count()
314

---

## Check find query non INDEX

titanic> db.person.find({Sex:'male'}).explain('executionStats')
--stage: 'COLLSCAN',  works: 888, advanced: 573, needTime: 314,

```
{
explainVersion: '1',
queryPlanner: {
namespace: 'titanic.person',
indexFilterSet: false,
parsedQuery: { Sex: { '$eq': 'male' } },
queryHash: '20F1BA5F',
planCacheKey: '20F1BA5F',
maxIndexedOrSolutionsReached: false,
maxIndexedAndSolutionsReached: false,
maxScansToExplodeReached: false,
winningPlan: {
stage: 'COLLSCAN',
filter: { Sex: { '$eq': 'male' } },
direction: 'forward'
},
rejectedPlans: []
},
executionStats: {
executionSuccess: true,
nReturned: 573,
executionTimeMillis: 0,
totalKeysExamined: 0,
totalDocsExamined: 887,
executionStages: {
stage: 'COLLSCAN',
filter: { Sex: { '$eq': 'male' } },
nReturned: 573,
executionTimeMillisEstimate: 0,
works: 888,
advanced: 573,
needTime: 314,
needYield: 0,
saveState: 0,
restoreState: 0,
isEOF: 1,
direction: 'forward',
docsExamined: 887
}
},
command: { find: 'person', filter: { Sex: 'male' }, '$db': 'titanic' },
serverInfo: {
host: 'mongodb',
port: 27017,
version: '6.0.7',
gitVersion: '202ad4fda2618c652e35f5981ef2f903d8dd1f1a'
},
serverParameters: {
internalQueryFacetBufferSizeBytes: 104857600,
internalQueryFacetMaxOutputDocSizeBytes: 104857600,
internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
internalDocumentSourceGroupMaxMemoryBytes: 104857600,
internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
internalQueryProhibitBlockingMergeOnMongoS: 0,
internalQueryMaxAddToSetBytes: 104857600,
internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
},
ok: 1
}
```

## Check find query with INDEX

db.person.createIndex({Sex:0})
--index sex_1
titanic> db.person.find({Sex:'male'}).explain('executionStats')
--stage: 'IXSCAN', indexName: 'Sex_1', works: 574, advanced: 573, needTime: 0,

```
{
	  explainVersion: '1',
	  queryPlanner: {
		namespace: 'titanic.person',
		indexFilterSet: false,
		parsedQuery: { Sex: { '$eq': 'male' } },
		queryHash: '20F1BA5F',
		planCacheKey: '464C398B',
		maxIndexedOrSolutionsReached: false,
		maxIndexedAndSolutionsReached: false,
		maxScansToExplodeReached: false,
		winningPlan: {
		  stage: 'FETCH',
		  inputStage: {
			stage: 'IXSCAN',
			keyPattern: { Sex: 1 },
			indexName: 'Sex_1',
			isMultiKey: false,
			multiKeyPaths: { Sex: [] },
			isUnique: false,
			isSparse: false,
			isPartial: false,
			indexVersion: 2,
			direction: 'forward',
			indexBounds: { Sex: [ '["male", "male"]' ] }
		  }
		},
		rejectedPlans: []
	  },
	  executionStats: {
		executionSuccess: true,
		nReturned: 573,
		executionTimeMillis: 1,
		totalKeysExamined: 573,
		totalDocsExamined: 573,
		executionStages: {
		  stage: 'FETCH',
		  nReturned: 573,
		  executionTimeMillisEstimate: 0,
		  works: 574,
		  advanced: 573,
		  needTime: 0,
		  needYield: 0,
		  saveState: 0,
		  restoreState: 0,
		  isEOF: 1,
		  docsExamined: 573,
		  alreadyHasObj: 0,
		  inputStage: {
			stage: 'IXSCAN',
			nReturned: 573,
			executionTimeMillisEstimate: 0,
			works: 574,
			advanced: 573,
			needTime: 0,
			needYield: 0,
			saveState: 0,
			restoreState: 0,
			isEOF: 1,
			keyPattern: { Sex: 1 },
			indexName: 'Sex_1',
			isMultiKey: false,
			multiKeyPaths: { Sex: [] },
			isUnique: false,
			isSparse: false,
			isPartial: false,
			indexVersion: 2,
			direction: 'forward',
			indexBounds: { Sex: [ '["male", "male"]' ] },
			keysExamined: 573,
			seeks: 1,
			dupsTested: 0,
			dupsDropped: 0
		  }
		}
	  },
	  command: { find: 'person', filter: { Sex: 'male' }, '$db': 'titanic' },
	  serverInfo: {
		host: 'mongodb',
		port: 27017,
		version: '6.0.7',
		gitVersion: '202ad4fda2618c652e35f5981ef2f903d8dd1f1a'
	  },
	  serverParameters: {
		internalQueryFacetBufferSizeBytes: 104857600,
		internalQueryFacetMaxOutputDocSizeBytes: 104857600,
		internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
		internalDocumentSourceGroupMaxMemoryBytes: 104857600,
		internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
		internalQueryProhibitBlockingMergeOnMongoS: 0,
		internalQueryMaxAddToSetBytes: 104857600,
		internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
	  },
	  ok: 1
	}
```

### FINAL

--DROP collection
db.person.drop();
--true

--DROP DB
db.dropDatabase()
--{ "ok" : 1 }