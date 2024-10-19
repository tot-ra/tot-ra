четверг, 14 мая 2009 г. в 16:34:49

Regio.ee - ведущая компания картографических и геодезических работ в Эстонии. У них очень неплохой сервис карт типа google maps отличающийся большей точностью. Хоть и работает на flash. Для моего проекта [terrideal.com](http://terrideal.com/) возникла задача переноса координат которые используются в regio на координаты google maps.

Как оказалось Regio использует коническую проекцию [Lambert-EST](http://en.wikipedia.org/wiki/Lambert_conformal_conic_projection) введённую в советское время для хорошей точности на местности. Google же использует мировую геодезическую систему [WGS84](http://en.wikipedia.org/wiki/World_Geodetic_System). Существует [конвертатор](http://kaardid.regio.ee/coords/index.php) на самом сайте regio, но понятное дело хотелось собственный вариант. Сначала я методом тыка пришёл к приблизительной формуле:

`wgs_x= 57.0014 + (est_x-6317802.5) / 111368; wgs_y= 24.0314 + (est_y-500000) / 59097.825;`

Но точность оставала желать лучшего. И вот наткнулся на [идеальный вариант](http://www2.regio.ee/alex/API2/test/gmapSyncedWithFT.html) написанный Саней Смирновым из Regio. Пришлось порыться в коде и найти нужную функцию с переносом в php. Там же можно найти и обратное направление и конвертирование в систему Меркатора.

```
/**
 * Converts Estonian L-EST coordinates to World Geodetic System
 *
 * @param float $X
 * @param float $Y
 * @return array
 */
function Est2Wgs($X, $Y) {
    
    $_PI= 3.14159265358979323846;
    $_n = 0.85417585805;
    $_x0= 6375000;
    $_y0= 500000;
    $_a = 6378137;
    $_F = 1.7988478514;
    $_e    = 0.0818191910428158;
    $_p0= 4020205.479;

    function sqr($a) { 
        return $a*$a; 
    }

    //$theta, $p, $t, $u;
    $Lo = 24 * pi() / 180;
    
    $ux = $X - $_y0;
    $uy = $Y - $_x0;
    
    $sx = $ux;
    $ux = $uy;
    $uy = $sx;
    
    $theta = atan($uy / ($_p0 - $ux));
    $tmpL = $theta / $_n + $Lo;
    
    $p = sqr($_p0 - $ux);
    $p += sqr($uy);
    $p = sqrt($p);
    
    $t = pow(($p / ($_a * $_F)), 1 / $_n);
    $u = pi() / 2 - 2 * atan($t);

    $tmpB = $u + 
    (sqr($_e)/2 + 5 * sqr($_e) * sqr($_e)/24 + pow($_e,6)/12 + 13 * pow($_e, 8)/360) * sin(2 * $u) + 
    (7 * pow($_e,4) /48  + 29 * pow($_e, 6)/240 + 811 * pow($_e, 8)/11520) * sin(4 * $u) + 
    (7 * pow($_e, 6)/120 + 81 * pow($_e, 8)/1120) * sin( 6 * $u);

    $X=rad2deg($tmpL);
    $Y=rad2deg($tmpB);

    return array($X, $Y);
}
```