  ----------------------------------------------------------------------------------------------------------------------------------------------------
  Repository                        Python %    Python     Domain         Primary purpose      Key dependencies     Architecture       Target use case
                                               primary?                                                             patterns           
  ------------------------------- ---------- ------------- -------------- -------------------- -------------------- ------------------ ---------------
  aiokafka (`aio-libs/aiokafka`)       93.1%      Yes      Distributed    Async Python client  async-timeout,       Async/await        Backend/data
                                                           streaming      for Apache Kafka --- packaging,           coroutine I/O;     engineers
                                                                          provides             typing-extensions,   layered Kafka      building
                                                                          `AIOKafkaProducer`   Cython (build),      wire-protocol;     real-time
                                                                          and                  cramjam (optional),  optional           event-driven
                                                                          `AIOKafkaConsumer`   gssapi (optional)    C-extension;       microservices
                                                                          integrated with                           consumer group     in async Python
                                                                          `asyncio`                                 coordinator;       (FastAPI,
                                                                                                                    pluggable codec    aiohttp)
                                                                                                                    registry           

  airbyte (`airbytehq/airbyte`)        51.3%      No       Data           ELT/ETL data         Kotlin/Gradle,       Polyglot platform; Data engineers
                                                           integration /  pipeline platform    airbyte-cdk          connector plugin   centralising
                                                           ELT            with 600+ connectors (Python), Docker,    model;             data from
                                                                          --- Python for       Pydantic             microservices;     APIs/DBs to
                                                                          CDK/connectors,                           low-code/no-code   warehouses
                                                                          Kotlin/Java for core                      CDK                
                                                                          runtime                                                      

  archivematica                        83.0%      Yes      Digital        Standards-based      Django, Celery,      Django MVC (MTV);  Archives,
  (`artefactual/archivematica`)                            preservation   digital preservation MySQL, FITS/JHOVE,   distributed task   libraries,
                                                                          system (OAIS/BagIt)  FFmpeg,              pipeline; FPR;     museums,
                                                                          --- ingest,          pytest-django        command pattern;   government
                                                                          normalise, and store                      storage            agencies
                                                                          digital objects                           abstraction        needing
                                                                                                                                       long-term
                                                                                                                                       digital
                                                                                                                                       preservation

  beets (`beetbox/beets`)              96.2%      Yes      Music library  CLI music library    mutagen,             Plugin/hook event  Music
                                                           management     manager and          musicbrainzngs,      system; CLI        enthusiasts and
                                                                          MusicBrainz          jellyfish, confuse,  subcommand         developers
                                                                          auto-tagger ---      requests, pyyaml     framework;         managing
                                                                          catalogs, corrects                        SQLite-backed ORM; personal music
                                                                          metadata, organises                       path-template      collections
                                                                          files                                     engine; autotag    
                                                                                                                    pipeline           

  MetaGPT                              97.5%      Yes      Multi-agent AI Multi-agent LLM      openai SDK,          Role-based         AI researchers
  (`FoundationAgents/MetaGPT`)                             / LLM          framework modelling  pydantic, tenacity,  multi-agent        and developers
                                                                          a software company   aiohttp, loguru,     system; SOP-driven building
                                                                          --- agents           gitpython, faiss     action graph;      multi-agent LLM
                                                                          collaborate to       (optional)           action             applications
                                                                          generate software                         abstraction;       
                                                                          artefacts                                 async-first;       
                                                                                                                    memory hierarchy   
  ----------------------------------------------------------------------------------------------------------------------------------------------------
