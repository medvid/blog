---
title: Funny busybox unicode text
date: 2015-02-02
aliases:
    - /busybox/fun/2015/02/02/busybox/
---

While trying to find out why busybox testsuite fails
(don't ask me why I was doing it), I noticed a funny test:

    echo -ne '' >input
    echo -ne 'The Andromeda Galaxy (pronounced /ænˈdrɒmədə/, also known as Messier 31, M31, or NGC224; often referred to as the Great Andromeda Nebula in older texts) is a spiral galaxy approximately 2,500,000 light-years (1.58×10^11 AU) away in the constellation Andromeda. It is the nearest spiral galaxy to our own, the Milky Way.
    Галактика або Туманність Андромеди (також відома як M31 за каталогом Мессьє та NGC224 за Новим загальним каталогом) — спіральна галактика, що знаходиться на відстані приблизно у 2,5 мільйони світлових років від нашої планети у сузір'ї Андромеди. На початку ХХІ ст. в центрі галактики виявлено чорну дірку.' | fold -sw66
    PASS: fold -sw66 with unicode input

It turns out that random snippet from Ukrainian wiki is the best way to test how fold utility handles unicode input...

P.S. Failing test is:

    echo -ne 'strstr\n' >input
    echo -ne '' | grep -w ^str input
    FAIL: grep -w ^str doesn't match str not at the beginning
    --- expected
    +++ actual
    @@ -0,0 +1 @@
    +strstr
