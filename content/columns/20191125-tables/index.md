---
title: "麻雀大会でどのように席を入れ替えれば毎回別の人と対戦できるか？"
url: "/p/vwkycpy"
date: "2019-11-25"
tags: ["コラム"]
---

16人で麻雀大会を開きたいと思います。
さて、せっかくなので、全員が毎回別のプレイヤーと同卓できるようにしたいです。
卓は 4 卓として、プレイヤー 1～16 がゲームごとにどのように座れば、一度も重複なく別のプレイヤーと同卓できるでしょうか？

{{< image w="500" src="tables.png" >}}

一例として、次のように座っていけば、5 ゲームで全員が他の人と対戦できます。
かつ、5ゲームの間は、一度も対戦相手が被ることはありません。

```
1ゲーム目
  1番卓:  1,  2,  3,  4
  2番卓:  5,  6,  7,  8
  3番卓:  9, 10, 11, 12
  4番卓: 13, 14, 15, 16

2ゲーム目
  1番卓:  1,  5,  9, 13
  2番卓:  2,  6, 10, 14
  3番卓:  3,  7, 11, 15
  4番卓:  4,  8, 12, 16

3ゲーム目
  1番卓:  1,  6, 11, 16
  2番卓:  2,  5, 12, 15
  3番卓:  3,  8,  9, 14
  4番卓:  4,  7, 10, 13

4ゲーム目
  1番卓:  1,  7, 12, 14
  2番卓:  2,  8, 11, 13
  3番卓:  3,  5, 10, 16
  4番卓:  4,  6,  9, 15

5ゲーム目
  1番卓:  1,  8, 10, 15
  2番卓:  2,  7,  9, 16
  3番卓:  3,  6, 12, 13
  4番卓:  4,  5, 11, 14
```

幹事の人は、次のような座席入れ替え表を作っておいて、くじ引きなどで決めてあげれば OK。

- {{< file src="tables.xlsx" caption="座席入れ替え表（くじ）" >}}

ちなみに、人数を増やして 20人（5卓）にした場合は、次のように 4 ゲームの間は対戦相手がかぶらないようにできますが、5 ゲーム目以降はかならず 1 度対戦した人と同卓になります。
この傾向はさらに人数を増やしても同じです。

```
1ゲーム目
  1番卓:  1,  2,  3,  4
  2番卓:  5,  6,  7,  8
  3番卓:  9, 10, 11, 12
  4番卓: 13, 14, 15, 16
  5番卓: 17, 18, 19, 20

2ゲーム目
  1番卓:  1,  5,  9, 13
  2番卓:  2,  6, 10, 17
  3番卓:  3,  7, 14, 18
  4番卓:  4, 11, 15, 19
  5番卓:  8, 12, 16, 20

3ゲーム目
  1番卓:  1,  6, 12, 18
  2番卓:  2,  5, 15, 20
  3番卓:  3,  9, 16, 19
  4番卓:  4,  7, 10, 13
  5番卓:  8, 11, 14, 17

4ゲーム目
  1番卓:  1, 10, 14, 19
  2番卓:  2,  7, 11, 16
  3番卓:  3,  6, 13, 20
  4番卓:  4,  5, 12, 17
  5番卓:  8,  9, 15, 18
```

**16人で大会を行った場合は、5ゲームで全員と重複せずに試合ができるのでちょうどいい人数**だと言えそうです。
あと、64人でやった場合は、21ゲームも重複せずに試合できるということが分かりました（すごぃ）。

この組み合わせは手で書き出していると面倒だったので、Python スクリプトで作成してみました。

{{< code lang="python" title="tables.py" >}}
PLAYERS = 16
TABLES = PLAYERS // 4

# matched[i] は i 番目の人が対戦済みの人のセット
matched = [set() for _ in range(PLAYERS)]

# seats[i] は i 番目の卓に座っている人のリスト
seats = [[] for _ in range(TABLES)]

# player が table に座れるか？
def can_seat(player, table):
    # 満席か？
    if len(seats[table]) == 4: return False
    # 対戦済みの人がいるか？
    return not(matched[player] & set(seats[table]))

# 1つ前の卓が空いているのに、次の卓に座ろうとしたら終わり
#（残りは卓の入れ替えでしかないので）
def no_more():
    prev_not_seated = False
    for i in range(TABLES):
        seated = len(seats[i]) > 0
        if seated and prev_not_seated:
            return True
        prev_not_seated = not(seated)
    return False

# 確定した席で対戦済み表を更新
def update_matched():
    for s in seats:
        for p in s:
            matched[p] |= set(s)

# 座席表を出力
def print_seats(game):
    print("{}ゲーム目".format(game))
    game += 1
    for i in range(TABLES):
        s = ', '.join(map(lambda p: "{:2}".format(p+1), seats[i]))
        print('  {}番卓: {}'.format(i + 1, s))
    print()

# player が座れる卓に座る
def take_seat(player, game):
    if no_more():
        return False

    # 全員座ったら出力して終了
    if player == PLAYERS:
        print_seats(game)
        update_matched()
        return True

    # 入れる卓に入る
    for t in range(TABLES):
        if can_seat(player, t):
            # 次のプレイヤーへ
            seats[t].append(player)
            complete = take_seat(player + 1, game)
            seats[t].remove(player)
            if complete: return True

if __name__ == '__main__':
    game = 1
    while take_seat(0, game):
        game += 1

{{< /code >}}

なんか、余計面倒だったような。。。
きっとうまく書いたら数行で書けたりするんだろうなぁ。

