Dump
```
sudo -u postgres pg_dump portal_asim | gzip > /database/backupdir/portal.sql.gz
sudo -u postgres pg_dump portal_asim_middle | gzip > /database/backupdir/portal_middle.sql.gz
```

Import
```
sudo -u postgres psql -d postgres -c "DROP DATABASE IF EXISTS portal;"
sudo -u postgres psql -d postgres -c "DROP DATABASE IF EXISTS portal_middle;"

sudo -u postgres createdb portal
sudo -u postgres createdb portal_middle

gunzip -c /database/backupdir/portal.sql.gz | sudo -u postgres psql -d portal
gunzip -c /database/backupdir/portal_middle.sql.gz | sudo -u postgres psql -d portal_middle
```
