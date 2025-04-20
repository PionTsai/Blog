# 玩家角色移動

最近在玩 GameMaker Studio 2，這是一款做遊戲的遊戲(X  
如果做出來的成品沒有要賣，基本上是免費的  
另外這款有上架 [Steam](https://store.steampowered.com/app/1670460/GameMaker/)  
所以我都是在買遊戲的遊戲裡面玩做遊戲的遊戲(X

遊戲時數到目前有近二十小時了  
雖然有不少時間是掛著然後查資料  
但扣除查資料跟環境設定  
我的進度確實是頗緩慢的  
到目前為止都還在處理所謂的「腳色移動」  
其理由就是每一部教學影片教的移動方式都不同  
沒錯，看了三部就有三種  
這次就想把它紀錄一下

## 共同部分

這次講的是簡單的操作方式  
也就是用鍵盤四個方向鍵進行二軸移動  
可以是上下左右，也可以是WASD  
三家在這方面做的事都一樣：

- 建立玩家物件(object)，我這邊取名為 `obj_player`
- 選擇 `obj_player` 的 sprite。sprite 是電腦圖學或遊戲製作領域的專有名詞，這可以單獨開一篇來解釋。我資歷尚淺，目前理解就是只要是一張圖（或一套動畫）就是一個 sprite
- 建立 create 腳本，這是物件初始化會執行的程式碼，整個生命週期就只會執行這一次。我是當作 constructor 來理解。寫下如下指令，基本上都是初始化變數。
  ```
  xspd = 0;
  yspd = 0;

  move_spd = 1;
  ```
- 先不考慮你想做成像初代仙劍是走斜線（還有人知道我在講啥嗎X）的模式，而是常見 RPGMaker 類型那樣走直角，那`xspd`與`ysd`就分別是橫向與縱向速度，然後`move_spd`是初始速度常數。GML 好像沒有`const`這種鎖定常數值的宣告。
- 接著建立 step 腳本，上面會有個腳印的圖標，意指移動規則就寫在這。step 還有 begin/end 的選項，初學者就先別管（我是覺得這些有種遊戲王在每個階段問有沒有事的感覺X）。寫下：
  ```
  var r_key = keyboard_check(vk_right);
  var l_key = keyboard_check(vk_left);
  var u_key = keyboard_check(vk_up);
  var d_key = keyboard_check(vk_down);

  var dx = r_key - l_key;
  var dy = d_key - u_key;

  xspd = dx * move_spd;
  yspd = dy * move_spd;
  ```
- 雖然各家變數名稱可能不同，但邏輯是一模一樣。這裡的變數要用`var`宣告，稱為「局部變數」，意指只在這份腳本內生效的變數。前面 create 寫的叫「實體變數」，會在整個物件內(`obj_player`)生效。[1](https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Overview/Variables_And_Variable_Scope.htm "關於變數的定義及種類可參考")
- `keyboard_check()`會檢查某特定按鍵是否被按下，並回傳True/False。所以按右時`dx > 0`，按左`dx < 0`；按下`dy > 0`，按上`dy < 0`(注意y軸座標往下才是正值)。如果要用WASD方向鍵就在對應位置輸入`"W"`, `"A"`, `"S"`, `"D"`即可。
- 確定方向之後，就乘上之前的速度常數`move_spd`就是你的水平、垂直速度了。然而，到這裡你的人物還無法實際動起來，因為這邊算的都是你自己建立的變數，跟系統本身沒有任何關係。之後不同家有不同的做法...


## 第一種：直接改變玩家位置

來自[影片清單](https://www.youtube.com/watch?v=KnfQo32ME5g&list=PL14Yj-e2sgzySnBUlQLhq2VJXRLi66gFf)，續之前的step，寫下
```
x += xspd;
y += yspd;
```
這裡的`x`, `y`是「內建變數」，就好比說底層有個像`class object`然後有定義這些參數跟它們的行為等等。那`x`, `y`就是玩家這物件的橫坐標與縱坐標。於是你就加上每單位時間(step)的速度，那不就動起來了嗎，很國中物理吧。

### 障礙物(碰撞)處理

如果地圖上有東西你希望玩家會被其阻擋，像是牆或樹之類的。首先你要先在地圖那邊放置物件(object)或地圖塊(tiles)接著在 step 寫下：
```
if place_meeting(x + xspd, y, <object>) == true {
	xspd = 0
}
if place_meeting(x, y + yspd, <object>) == true {
	yspd = 0
}

x += xspd;
y += yspd;
```
`place_meeting()` 回傳玩家「移動後」是否碰到物體，True 表示碰到，那就把速度降為零，然後橫軸跟縱軸各做一次。這必須寫在實際修改座標之前，畢竟你都穿過去了才問有沒有碰撞就太遲了。

額外補充：若障礙物是地圖塊，假設說我在地圖上的一個圖層加上牆、障礙物，命名為`Tiles_collision`，然後在 create 加上一行
```
collide_tile = layer_tilemap_get_id("Tiles_collision");
```
然後判斷式改為 `if place_meeting(x + xspd, y, collide_tile)` 即可。


## 第二種：改變玩家的速度

出自[影片](https://www.youtube.com/watch?v=M4kxcAc2df0)。同樣，續共通的 step，寫下
```
hspeed = xspd;
vspeed = yspd;
```

`hspeed`, `vspeed`也是內建變數，但會直接設定物件的移動速度。所以在每個step都更新它的這兩個速度，就會照著這速度動起來了。

### 障礙物(碰撞)處理

```
hspeed = xspd;
vspeed = yspd;

if place_meeting(x + hspeed, y, <object>) == true {
	hspeed = 0
}
if place_meeting(x, y + vspeed, <object>) == true {
	vspeed = 0
}
```
類似第一種作法，但修改速度參數要放在前面，不然就是你處理完碰撞了，結果馬上又把速度還原，等於做白工。

## 第三種：move_and_collide()

這是官方頻道出[教學影片](https://www.youtube.com/watch?v=1J5EydrnIPs)提供的做法(官方的竟然最晚看的，而且這也三部影片中最晚發的)。在 step 寫下：
```
move_and_collide(xspd, yspd, collide_tile);
```
這個函數其實有很多可選輸入，但必要的前三個分別是水平速度、垂直速度、碰撞判定物。接著引擎就會處理玩家移動、障礙物阻擋的處理，一次搞定。第三個輸入可以是物件也可以是地圖塊等各種東西，但我不知道如果同時要處理多種類的碰撞該怎麼輸入或其他辦法，以後學到再說。

## 差異

首先，這三者當然是不能同時使用，人物的速度會疊加。  
那如果你是比較好奇、或者喜歡挖坑、或這像我要給這篇明明是基礎卻要寫些很深奧的內容的，就會想要知道這三者之間會怎樣交互作用。  
比如說第二種`hspeed`, `vspeed`很顯然只要非零就會影響座標`x`, `y`，那反過來第一種直接修改`x`, `y`的位置以及第三種用函數會不會影響`hspeed`, `vspeed`的值呢？  
我測試之後，看起來是否定的。例如我在 step 中寫（省略開頭讀取按鍵的部分）
```
xspd = dx * move_spd;
yspd = dy * move_spd;

x += xspd;
y += yspd;

if (hspeed == xspd) {
	hspeed = 3 * xspd;
}
```
意思是如果我水平移動的`hspeed`有增加，那人物就會以三倍速水平飛出去。  
但並沒有，依然是一倍速。  
如果 if 條件改成 `hspeed == 0`呢？也就是理論上你停止水平移動，那你反而會飛出去。  
這時遊戲剛開啟時你是靜止的，但一動就馬上飛走。也就是你雖然動了，但系統看到你的`hspeed`還是零，所以你就三倍速了。  
也就是說，遊戲引擎沒有聰明到在你位移的同時幫你計算速度。

接著，把修改`x`, `y`的那兩行改成call `move_and_collide()`，結果還是一樣，只是遇到障礙物時會緩速(兩倍速，假設沒有其他處理碰撞的腳本)。  
因此我猜測`move_and_collide()`內部的移動方法也是類似第一種直接對座標下手。

我記得爬文時看過有建議玩家下的移動指令是用改變座標`x`, `y`的方式，而不是設`vspeed`, `hspeed`。  
我以這種思維衍生出的應用情境是：玩家主動移動以及+跑速的效果都是用前者，而環境的影響（例如強風導致往某方向飄移）用後者。當然，我並沒什麼經驗可以判斷這種做法好不好。

## 斜線？

如果同時按上下鍵之一、左右鍵之一，你就會同時有水平跟垂直速度。這時你會注意到，走斜線的速度似乎會比較快？
因為你的水平跟垂直預設速度都是 1，所以同時存在時，你的斜線速度就會是兩條向量相加，也就是斜 45 度 $\sqrt2$ 倍速度。
可以做個簡單的修正：
```
// Walking diagonally
if (dx != 0 and dy != 0) {
	var scale = sqrt(dx * dx + dy * dy);
	dx /= scale;
	dy /= scale;
}
```
然而，這時就會出現另一個問題...  
如果你有設好鏡頭跟隨玩家，你會發現當你走斜線時，畫面會開始震動。如果畫面解析度小、像素太大會更嚴重。  
這是因為鏡頭是以每單位時間（每幀？）進行點陣圖渲染成像。當腳色水平、垂直移動時，每單位時間位移距離都是整數，鏡頭跟腳色像素可以對齊，看起來就很平滑。然而當腳色移動座標出現小數（因為除以 $\sqrt2$）時，鏡頭渲染就會跟腳色座標產生誤差，只有當腳色經過整數位置時才會成像，因此就會出現鋸齒般的畫面震動。  
比較簡單粗暴的做法是精化像素，例如原本一顆像素用四顆表示，或者直接提高解析度；但顯然都不是很完美的做法。另一種做法是手動設置鏡頭，關鍵字 pixel art Camera，這看起來又是個坑，目前我也還沒完全搞懂，正在看[影片](https://www.youtube.com/watch?v=5hTYexed8-Q)研究中。未來有機會再發一篇吧。


## 小結

結果我為了寫一個別人可能花3~5分鐘就帶過的內容，就花了將近一整天...  
甚至都還沒講解如何讓腳色在面朝不同方向使用不同方向的 sprite，以及如何在移動跟靜止時分別用對應的 sprites (這樣就有 4 * 2 = 8 個 sprites)。實作很簡單，也沒有像這次講的有這麼多變化，但要做出一些小效果也是花了我不少時間。一樣有空會寫一篇。