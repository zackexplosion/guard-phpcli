![Imgur](http://i.imgur.com/VN3TPrD.png)

這幾天因為工作需求，需要將程式重新使用PHP開發，因為習慣Rails的開發流程後再回去寫傳統的PHP，總是覺得不太順手。Rails可以搭配[Guard](/p/2)，達到一存檔就讓程式重新跑的效果，但是據我所知，目前PHP沒有搭配的工具可以達到監控檔案目錄讓程式重新跑的效果。

### 找不到工具，那就自己做吧!

既然兩種語言都學過了，就把他們混合起來用吧!，需求其實很簡單，就只有下面幾點而已

* 監控所有**.php**附檔名的檔案
* 當檔案變更時自動下**php test.php**的指令
* 顯示結果到終端機

### PHP測試CODE

首先，先準備一份測試用的PHP

**指令**

``` bash
mkdir guard-php
cd guard-php

vim test.php
```

**test.php**

``` php
<?php
	echo time();
?>
```

範例中的CODE非常簡單，單純echo現在的時間而已，同時也方便我們稍後來檢查Guard有沒有順利執行

### 設定 Gemfile 內容


指令

``` bash
touch Gemfile
vim Gemfile
```

**Gemfile**

``` ruby
source 'https://rubygems.org'
gem 'guard'
```

Gemfile中相當的單純，因為只有用到Guard而已XDD

### Guardfile

``` bash
guard init
```

產生Guardfile，因為我們沒有裝任何Guard的Pulgin，所以Guardfile產生後只會有一堆註解而已。接著重頭戲來啦，透過inline guard的方式，我們可以直接在Guardfile裡面寫上客製化的執行工作。

**Guardfile**

``` ruby
module ::Guard
  class Phpcli < Plugin
    def run_all
    end

    def run_on_modifications(paths)
      puts nil # 為了讓輸出好看一點XD
      puts `php test.php`
    end
  end
end

guard 'phpcli' do
  watch(%r{\.php})
end
```

宣告一個新的class叫做Phpcli，修改**run_on_modifications**這個action，裡面就是每次Guard監測到檔案變更時要執行的工作，如同剛剛的需求，我希望每次存檔時就幫我跑```php test.php```的指令。最後三行是在告訴Guard說要監控哪些檔案的狀態，監控到該告訴哪個module檔案有變更了。

最後，啟動Guard來實驗一下剛剛的流程有沒有錯誤

``` bash
guard
```

存取一下test.php後會發現Guard有跑了！

```
➜  guard-php  guard
14:51:54 - INFO - Guard is now watching at '/Users/zack/projects/guard-php'
1432882319%                                                                                                                                                         ➜  guard-php
```

不過怎麼只有跑一次呀XDDDDDDDDDDDDDDD


後來發現是因為使用exec去呼叫shell會讓目前的執行續中斷，將guardfile中的exec換成用**`**(backtricks)就可以了

``` ruby
module ::Guard
  class Phpcli < Plugin
    def run_all

    end

    def run_on_modifications(paths)
      puts nil # 為了讓輸出好看一點XD
      puts `php test.php`
    end
  end
end

guard 'phpcli' do
  watch(%r{\.php})
end
```






#### 參考資料

* <https://github.com/guard/guard/wiki/Create-a-guard>
* <http://guardgem.org/>
* <http://tech.natemurray.com/2007/03/ruby-shell-commands.html>
