# TeslaMate Portainer Database Restore

1. from old machine, obtain teslamate.bck using method in https://docs.teslamate.org/docs/maintenance/backup_restore and place into machine that host portainer
2. create new stack from official docker-compose.yml https://docs.teslamate.org/docs/installation/docker, make sure all things running
3. ssh into machine that host portainer
4. find out container id

```
#---find out container id of teslamate-database-1
docker container ls

#---copy teslamate.bck into container
docker cp ./teslamate.bck 46eded4ad7b3:/tmp/teslamate.bck

```

5. goto console in teslamate-database-1
	1. From Portainer GUI, Select "Containers"
	2. Select "teslamate-database-1"
	3. Select "Console" , then "Connect"
	4. Command prompt should shows up
6. run clean up and reinitialize query from https://docs.teslamate.org/docs/maintenance/backup_restore
>[!note]
>We are using just psql command without docker command since we're already in the docker instance
>
>Correct output should shows bunch of output with lot of activity such as DROP, CREATE, ETC
```
root@46eded4ad7b3:/# psql -U teslamate << .  
drop schema public cascade;  
create schema public;  
create extension cube;  
create extension earthdistance;  
CREATE OR REPLACE FUNCTION public.ll_to_earth(float8, float8)  
RETURNS public.earth  
LANGUAGE SQL  
IMMUTABLE STRICT  
PARALLEL SAFE  
AS 'SELECT public.cube(public.cube(public.cube(public.earth()*cos(radians(\$1))*cos(radians(\$2))),public.earth()*cos(radians(\$1))*sin(radians(\$2))),public.earth()*sin(radians(\$1)))::public.earth';  
.
```
7. Run the import command
>[!note]
>We are using just psql command without docker command since we're already in the docker instance
>
>Correct output should shows bunch of output with lot of activity such as SET CREATE, ALTER, etc
```
cd /tmp
psql -U teslamate -d teslamate < teslamate.bck
```

8. restart all stack and check in grafana dashboard
