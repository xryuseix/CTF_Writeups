# Bait:OSINT:pts

There is an IP Camera close to the location where the picture was taken, find the IP of that camera.

Note: Please put the IP in the correct format for submission, like this:
SBCTF{127.0.0.1}

![](./images/bait.png)

# Solution

まずは看板を見て，このように検索をします．「"282-8233" boat」

[このサイト](https://bestadventurespots.com/places-you-can-dock-eat-in-cape-coral/)にレストラン名(Miceli’s Restaurant)が書かれていました．

Google Mapから[場所が特定できます](https://www.google.com/maps/@26.6390423,-82.0585524,3a,75y,325.61h,89.17t/data=!3m6!1e1!3m4!1siLfTFvNtz5MbhsvmLqVPzQ!2e0!7i16384!8i8192)．

近くのIPカメラはトラフィックカメラだと推測し，検索すると[見つかります](https://www.traffic-cams.com/cape-coral/florida/all)．

このサイトで先程のMiceli's Restaurantの場所を調べると3台あり，ページを開くと[このサイトのリンク](www.pineislandbait.com)が書かれています．サイトの「Live Webcam 3」を開発者モードで開きます．以下のリンクにアクセスしているそうです．

[http://ddbait.dyndns.org:8090](http://ddbait.dyndns.org:8090)

最後にwhoisでIPアドレスを特定します．

## SBCTF{73.156.111.21}