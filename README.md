"nginx-reverseproxy"는 동일한 서버에서 여러 앱을 호스팅하기 위한 간단한 리버스 프록시입니다. 이 프록시는 포트 80(또는 HTTP2 구현을 사용하는 경우 443)에서 듣고, 제공된 도메인에 따라 사용자를 특정 포트로 리디렉션하여 해당 포트에서 실행 중인 Node 앱으로 연결합니다.

이 구현은 앱 간에 독립성을 유지하여 한 앱의 오작동이 다른 앱에 영향을 미치지 않도록 합니다. 앱은 독립적으로 일시 중지, 재시작 또는 업데이트할 수 있습니다. 프록시 서버에 대한 변경 사항은 서버가 연결할 수 없더라도 애플리케이션 동작에 영향을 미치지 않으며 온라인 상태를 유지합니다.

사용법:

서버에 Nginx를 설치합니다 (apt를 사용하는 예제):

shell
Copy code
sudo apt-get install nginx
기본 서버 블록 구성 파일을 편집합니다:

shell
Copy code
sudo nano /etc/nginx/sites-available/default
파일의 내용을 모두 삭제하고 새 프록시 서버를 포함시킵니다. HTTP 또는 HTTP2를 사용하려면 nginx/default 파일을 복사하십시오. HTTPS 설정에 대한 자세한 내용은 "HTTPS 설정" 섹션을 참조하십시오. 프록시 서버는 포트 80 또는 443에서 듣습니다.

이 예제에서는 세 개의 앱에 대한 프록시를 만듭니다. 각각은 다른 도메인에서 실행되며 (각 서버 블록의 server_name 값을 확인하십시오), 각각의 앱은 다른 proxy_pass 포트에서 실행됩니다. 각 앱이 proxy_pass에서 선택한 포트와 동일한 포트에서 실행되는지 확인합니다.

앱이 올바른 포트에서 실행되는지 확인한 후 Nginx 서비스를 다시 시작하여 변경 사항을 적용합니다:

shell
Copy code
sudo service nginx restart
HTTPS 설정:

dhparam.pem 파일을 생성합니다:

shell
Copy code
openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
Let's Encrypt를 사용하여 SSL 인증서를 생성하는 방법은 제공된 링크에서 자세히 설명하고 있습니다.

SSL 인증서를 갱신하는 방법:

수동으로 갱신:

shell
Copy code
sudo certbot renew --dry-run
자동으로 갱신하려면 cron 작업을 설정합니다:

shell
Copy code
sudo crontab -e
다음 두 줄을 삽입합니다:

shell
Copy code
30 2 * * 1 /usr/bin/certbot renew --dry-run
35 2 * * 1 /bin/systemctl reload nginx
Nginx 구성을 테스트하려면 다음을 사용할 수 있습니다:

shell
Copy code
sudo nginx -t
추가 정보:

사용자가 앱 포트에 직접 액세스하지 못하도록하려면 app.listen 명령에 호스트 이름을 추가해야 합니다. 이렇게 하면 프록시만 해당 포트에 액세스하여 사용자로 리디렉션할 수 있습니다.

javascript
Copy code
var listener = app.listen(port, 'localhost', function() {
    console.log("Listening on port " + listener.address().port);
});
