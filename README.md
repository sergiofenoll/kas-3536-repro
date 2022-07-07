# KAS-3536 Reproduction

Put a large file in `./data/files`. In this example I called it `aaa.zip`, but call it whatever you want. Note that if you pick a different name, you need to adapt the query below and the `./data/files/delta.json` file.

Run the following query against the database:

``` sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX nie: <http://www.semanticdesktop.org/ontologies/2007/01/19/nie#>
PREFIX dbpedia: <http://dbpedia.org/resource/>
PREFIX mu: <http://mu.semte.ch/vocabularies/core/>
PREFIX nfo: <http://www.semanticdesktop.org/ontologies/2007/03/22/nfo#>
PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>

INSERT DATA {
  GRAPH <http://mu.semte.ch/graphs/application> {
    <http://mu.semte.ch/services/file-service/files/7f8d220b-8e4c-44be-a352-6a815f36c990> a nfo:FileDataObject ;
      nfo:fileName "aaa.zip" ;
      mu:uuid "7f8d220b-8e4c-44be-a352-6a815f36c990" ;
      dct:format "application/zip" ;
      nfo:fileSize 1086236000 ;
      dbpedia:fileExtension "zip" ;
      dct:created "2023-07-06T14:38:37.886Z"^^xsd:dateTime ;
      dct:modified "2023-07-06T14:38:37.886Z"^^xsd:dateTime  .
    <share://aaa.zip> a nfo:FileDataObject ;
      nie:dataSource <http://mu.semte.ch/services/file-service/files/7f8d220b-8e4c-44be-a352-6a815f36c990> ;
      nfo:fileName "aaa.zip" ;
      mu:uuid "5ef88bbb-0e80-4d01-a658-4030f99fad37" ;
      dct:format "application/zip" ;
      nfo:fileSize 1086236000 ;
      dbpedia:fileExtension "zip" ;
      dct:created "2023-07-06T14:38:37.886Z"^^xsd:dateTime ;
      dct:modified "2023-07-06T14:38:37.886Z"^^xsd:dateTime  .
    
    <http://mu.semte.ch/services/file-service/files/d6e2dfa3-82dc-4ff0-b05d-a4ff2655de6e> a nfo:FileDataObject ;
      nfo:fileName "delta.json" ;
      mu:uuid "d6e2dfa3-82dc-4ff0-b05d-a4ff2655de6e" ;
      dct:format "application/json" ;
      nfo:fileSize 4000 ;
      dbpedia:fileExtension "json" ;
      dct:creator <http://redpencil.data.gift/services/ttl-to-delta-service> ;
      dct:created "2023-07-06T14:38:37.886Z"^^xsd:dateTime ;
      dct:modified "2023-07-06T14:38:37.886Z"^^xsd:dateTime  .
    <share://delta.json> a nfo:FileDataObject ;
      nie:dataSource <http://mu.semte.ch/services/file-service/files/d6e2dfa3-82dc-4ff0-b05d-a4ff2655de6e> ;
      nfo:fileName "delta.json" ;
      mu:uuid "5ef88bbb-0e80-4d01-a658-4030f99fad37" ;
      dct:format "application/json" ;
      nfo:fileSize 4000 ;
      dbpedia:fileExtension "json" ;
      dct:created "2023-07-06T14:38:37.886Z"^^xsd:dateTime ;
      dct:modified "2023-07-06T14:38:37.886Z"^^xsd:dateTime  .
    
   <document> ext:file <http://mu.semte.ch/services/file-service/files/7f8d220b-8e4c-44be-a352-6a815f36c990> ;
              ext:toegangsniveauVoorDocumentVersie <http://themis.vlaanderen.be/id/concept/toegangsniveau/c3de9c70-391e-4031-a85e-4b03433d6266> .
   <document2> ext:file <http://mu.semte.ch/services/file-service/files/d6e2dfa3-82dc-4ff0-b05d-a4ff2655de6e> ;
              ext:toegangsniveauVoorDocumentVersie <http://themis.vlaanderen.be/id/concept/toegangsniveau/c3de9c70-391e-4031-a85e-4b03433d6266> .
  }
}
```

You can now download the file from your browser or using cURL by fetching the following URLs:

- http://localhost/files/7f8d220b-8e4c-44be-a352-6a815f36c990/download This is via the regular identifier
- http://localhost:1080/files/7f8d220b-8e4c-44be-a352-6a815f36c990/download This is via the identifier with a higher `idle_timeout`

If the file you provided is large enough and takes long enough to download, fetching it via the first link should result in a failed/partial download, while the second link should keep going as expected (unless the file is so large that it exceeds the raised `idle_timeout`).

## With themis-publication-consumer

Replace `SYNC_BASE_URL` to point to the `identifier`'s container in the docker-compose file.

In `./data/files/delta.json`, change the string `share://aaa.zip` to match the name of the file you provided. Start the `consumer` service, it should immediately try to sync. It's set up to use the default identifier, in which case it should get to the point of trying to download the file, but hang after a while. `./data/files/consumed` will contain a partial copy of the file that you provided.

Replace `SYNC_BASE_URL` to point to `my-identifier` instead, reset the DB's graph (I do this via the conductor, but choose whatever you find suitable) and restart the consumer. This time you should see it succeed the download and continue logging the expected output. `./data/files/consumed` should contain an exact copy of the file that you provided.
