loads trials


```
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/rmarkbio/rd_speaker_influence/master/Tidy_Clinical_Trials.csv?token=AKOT4ZHTX3ZZCLLAPNVU4M2525LQQ' AS Row CREATE (T:Trial {Primary_NPI: toInteger(Row.NPI), Trial_Name: Row.Trial_Name, Co_investigators: Row.Co_investigators})
```

Loads doctors
```
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/rmarkbio/rd_speaker_influence/master/Doctor_data.csv?token=AKOT4ZDBHGJHY5DREEICFWC52VYAG' AS Line CREATE (d:doc {NPI: toInteger(Line.NPI), first_name: Line.first_name, last_name: Line.last_name, state: Line.state})
```

Loads Conferences
```
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/rmarkbio/rd_speaker_influence/master/Tidy_Data/Tidy_Conference.csv?token=AKOT4ZEIJMRLFQE7YEVTKVS56QB4C' AS row CREATE ( c:conf {Speaker: row.Speaker,  Attendee_List: row.Attendee_list, Conference_name: row.Conference_name, conference_date: row.conference_date})
```
Returns Doctors Segmented by State


```
MATCH (d:doc), (d2:doc) WHERE EXISTS (d.state) AND EXISTS (d2.state) AND d.state=d2.state CREATE (d)-[:works]->(d2);
```


Matches Trials with co-authors


```
// Doctor -> Co-authored -> Trial 
MATCH (T:Trial)
WITH T.Co_investigators as NPI, T
WITH substring(NPI, 1, size(NPI)-2) as NPIs, T
WITH split(NPIs, ',') as NPI_LIST, T
UNWIND NPI_LIST as co_inv
WITH co_inv, T
MATCH (d:doc)  WHERE d.NPI = toInt(co_inv)
CREATE (d)-[:Co_Authored]->(T);
```

Matches Trials with primary authors


```
//Doctor -> Conducted -> Trial

MATCH (d:doc), (T:Trial) WHERE d.NPI=T.Primary_NPI CREATE (d)-[:Conducted]->(T);```
```

Matches Conferences with Primary Speaker

```
//Doctor -> Spoke at -> conference
MATCH (d:doc), (c:conf)
WHERE d.NPI=toInt(c.Speaker)
CREATE (d)-[:Spoke]->(c);
```

Matches Conferences with non-speaker attendees 

```
//Doctor -> Attended -> Conf
MATCH (c:conf)
WITH c.Attendee_List as NPI, c
WITH substring(NPI, 1, size(NPI)-2) as NPIs, c
WITH split(NPIs, ',') as NPI_LIST, c
UNWIND NPI_LIST as co_inv
with co_inv, c
MATCH (d:doc)  WHERE d.NPI = toInt(co_inv)
CREATE (d)-[:attended]->(c);
```
Removes all nodes not in a relationship


```
MATCH (n)
WHERE size((n)--())=0
DELETE (n)
```


Merges all co-investigators into a single node

```
MATCH (T:Trial)
UNWIND T.Co_investigators as co_inv
MERGE (d:doc {NPI: co_inv})
MERGE (T)-[:AUTHORED_BY]->(d)
```

