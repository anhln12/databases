Trong C:\Users\Admin\product\19.0.0\client_1\network\admin tạo file tnsnames.ora

Có nội dung

```
INET_DEV_247 =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 14.248.80.24)(PORT = 11111))
 )
    (CONNECT_DATA =
    (SID = db11g)
    )
 )
 
IT_MAPI =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.192.104)(PORT = 11111))
 )
    (CONNECT_DATA =
    (SID = oradb)
    )
 )


NM_USSD =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 203.162.166.168)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = oradb)
    )
 )
 
 
IT_VSKILL =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.192.104)(PORT = 22203))
 )
    (CONNECT_DATA =
    (SID = vskill01)
    )
 )

IT_THUOC =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.192.104)(PORT = 22203))
 )
    (CONNECT_DATA =
    (SID = inetdb02_dr)
    )
 )

IT_VSKILL_IDC =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 203.162.166.177)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = vskill)
    )
 )



VX =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 103.21.150.231)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = bluezone)
    )
 )




 
ASIM_UAT_APIGW =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 10.253.60.13)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = telcouat01)
    )
 )

ASIM_APIGW_DR =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 172.30.9.35)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = oradb01)
    )
 )

ASIM_APIGW =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 172.30.9.36)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = oradr01)
    )
 )
 

ASIM_BSS =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 10.30.5.21)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = bss1)
    )
 )


ASIM_TOUP_DUYTAN =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 10.255.68.21)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = asimdb)
    )
 )
 
ASIM_API_C06 =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 10.30.21.11)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = oradb)
    )
 )

ASIM_API_PDB1_C06 =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 10.30.21.11)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SERVICE_NAME = pdb1)
    )
 )
 
 
ASIM_API_PDB1_C06_STANDBY =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 10.30.21.12)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SERVICE_NAME = pdb1)
    )
 )
 
ASIM_MACOM =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 10.255.60.11)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = macom)
    )
 )
 
ASIM_TOUP=
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 172.30.9.11)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SID = asimdb)
    )
 )
 
 
ASIM_BSS_UAT =
(DESCRIPTION =
   (ADDRESS_LIST =
     (ADDRESS = (PROTOCOL = TCP)(HOST = 10.253.60.13)(PORT = 1521))
 )
    (CONNECT_DATA =
    (SERVICE_NAME = telcouat01)
    )
 )


 
```
