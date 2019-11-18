loads trials



'''
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/rmarkbio/rd_speaker_influence/master/Tidy_Clinical_Trials.csv?token=AKOT4ZHTX3ZZCLLAPNVU4M2525LQQ' AS Row CREATE (T:Trial {Primary_NPI: toInteger(Row.NPI), Trial_Name: Row.Trial_Name, Co_investigators: Row.Co_investigators})
'''

Loads doctors
'''
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/rmarkbio/rd_speaker_influence/master/Doctor_data.csv?token=AKOT4ZDBHGJHY5DREEICFWC52VYAG' AS Line CREATE (d:doc {NPI: toInteger(Line.NPI), first_name: Line.first_name, last_name: Line.last_name, state: Line.state})
'''

Returns Doctors Segmented by State
'''
MATCH (d:doc), (d2:doc) WHERE EXISTS (d.state) AND EXISTS (d2.state) AND d.state=d2.state CREATE (d)-[:works]->(d2);
'''

Matches Trials with co-authors
'''
MATCH (T:Trial)
WITH T.Co_investigators as NPI
WITH substring(NPI, 1, size(NPI)-2) as NPIs
WITH split(NPIs, ',') as NPI_LIST
UNWIND NPI_LIST as co_inv
MATCH (d:doc)  WHERE d.NPI = toInt(co_inv)
CREATE (d)-[:Co_Authored]->(T);
'''

Matches Trials with primary authors


'''
MATCH (d:doc), (T:Trial) WHERE d.NPI=T.Primary_NPI CREATE (d)-[:Conducted]->(T);
'''

Removes all nodes not in a relationship
'''
MATCH (n)
WHERE size((n)--())=0
DELETE (n)
'''


Merges all co-investigators into a single node

'''
MATCH (T:Trial)
UNWIND T.Co_investigators as co_inv
MERGE (d:doc {NPI: co_inv})
MERGE (T)-[:AUTHORED_BY]->(d)
'''

