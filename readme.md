#1. 数据预处理
>提取地名、拍摄时间  

>事件  
>摄影师  

**一级标签**  
>['2016图编外拍', '2016外国摄影师拍北京-最终发回文件', '2017希腊摄影师拍北京-最终发回成片', '2017年外拍', '2018一带一路摄影师拍北京-最终发回成片', '2018图编外拍', '2018外拍', '航拍挑图', '航拍（改）-常旭', '2014年外国摄影师拍北京-643张', '2015年外国摄影师拍北京-1479张', '2018图编外拍 2', '2019图编外拍', '友协', '摄影师投稿作品']  
>一级标签共：15种

##1.1 提取时间  
>日期格式过于分散，无法找到统一的提取规则，只能大方向匹配能够提取的，然后对有的图片的时间进行单独标注  
>主要包括的的格式有：2012年6月3日，2012年06月03日，2012-2-12，2012-12-21，2013年6月，20120102，  
>采用了以下几种方式处理：  
> - 正则表达式匹配，匹配年月日格式和以‘-’连接的格式
> - 将标题切分后，统计指定字符长度的元素，如20120202这种八位的数字，然后提取具体数字  
>


##1.2 提取人名信息  
使用jieba分词，首先创建自定义的词典，很多标题中含有大量的重复性名词，可以添加之词典中，且方便进行词频的统计
通过关键词提取获取每个标题中的关键字，并获取关键词排序

    jieba.analyse.extract_tags(title, topK=3)
    ******************************
    桑浥  20160409  平谷  桃花节  挑  [  Originals  ]  桑浥  20160409  -  ISY  _  2178  .  j
    关键词提取： 桑浥/20160409/桃花节/Originals/ISY/2178/平谷
    关键词top3： 桑浥/20160409/桃花节
    总词数16
    从16 中取出2 个词
    关键词topk： 桑浥/20160409
    ******************************


2020年7月7日结果：

    ('0013341.jpg', ['2017年外拍', '2017', '4', '4', ['陈硕']])
    ('0006870.JPG', ['2016外国摄影师拍北京-最终发回文件', '2016', '05', '30', ['霍姆兹']])
    ('0013342.jpg', ['2018一带一路摄影师拍北京-最终发回成片', '2018', '5', '10', ['斯坦科', '立陶宛']])
    ('0013343.jpg', ['2018一带一路摄影师拍北京-最终发回成片', '2018', '5', '10', ['斯坦科', '立陶宛']])
    ('0013344.jpg', ['2018一带一路摄影师拍北京-最终发回成片', '2018', '5', '10', ['斯坦科', '立陶宛']])
    ('0013345.jpg', ['2018一带一路摄影师拍北京-最终发回成片', '2018', '5', '10', ['斯坦科', '立陶宛']])  
    
##1.3 提取关键词
通过jieba所实现的基于IT-IDF算法的函数进行关键字的提取，注意对关键字中作者名和时间信息的过滤，最后通过词性过滤的方法去除无效信息，保留关键信息
```python
    keyword_list = []
    keywords_top = jieba.analyse.extract_tags(title, topK=10)  # 关键词前10位，返回值为列表
    print('关键词top 10： {}'.format(keywords_top))
    for word in keywords_top:
        if word not in name and word.isalpha(): #去除人名和日期
            keyword_list.append(word)
    print('关键字：{}'.format(keyword_list))

    #根据词性过滤有效信息
    rule = ['ns', 'nt', 'nz', 'f', 'i', 'l', 'j', 'vn', 'n', 'Ng', 'nr', 'z']
    words = pseg.cut(repr(keyword_list), use_paddle=True)  # paddle模式
    result_key = []  # 最终返回信息
    for word, flag in words:
        if flag in rule:
            print(" word:{}  flag: {} ".format(flag, word))
            result_key.append(word)
    print('过滤后关键字：{}'.format(result_key))
    return result_key
```
7月8号结果  

    00000001.jpg	2016图编外拍	2016	11	18	['李晓尹']	['美丽乡村', '顺义马坡镇南卷村', '村委会', '中国']	
    00000010.jpg	2016图编外拍	2016	1	14	['李晓尹']	['左安门', '猴年造型', '角楼', '王府井', '中国']	
    00000011.jpg	2016图编外拍	2016	1	14	['李晓尹']	['王府井', '猴年造型', '合影留念', '猴子', '游客', '造型', '中国']		
    00000044.jpg	2016图编外拍	2016	1	1	['李晓尹']	['外国摄影师巡展五彩城', '中国']	
    00000072.jpg	2016图编外拍	2016	7	12	['李晓尹']	['画火车王忠良', '境外', '中国']	
    00000079.jpg	2016图编外拍	2016	7	16	['李晓尹', '惠民']	['科教', '香山', '中关村', '刺猬']
    00004942.JPG	2016图编外拍	2016	04	28	[]	['桑浥', '手绘', '开幕式', '北海', '胡同', '京城']
    00014887.JPG	2018外拍	2018	None	None	['常旭']	['艺术展', '荆山', '南方']
    00016947.JPG	航拍（改）常旭	2016	10	11	['任晓峰', '常旭', '马文晓']	['司马台长城']
    
##未完成功能
 - 关键词的词频统计

###PS
1.关于是否使用jieba.analyse.extract_tags的allowPOS属性来过滤词性  
>实验中证明使用该参数时运行速度会变慢，不如自己写个if语句来判断  

2.添加自定义字典时，如果字典内容过多会明显拖慢运行速度，因此需要使用高质量而非高数量的字典

3.使jieba支持特殊字符，如外国人的中文翻译名‘多纳塔斯•斯坦科维休斯’这种格式  
```
搜索
re_han_default = re.compile("([\u4E00-\u9FD5a-zA-Z0-9+#&\._]+)", re.U)
改成
re_han_default = re.compile("(.+)", re.U)
搜索
re_userdict = re.compile('^(.+?)( [0-9]+)?( [a-z]+)?$', re.U)
改成
re_userdict = re.compile('^(.+?)(\u0040\u0040[0-9]+)?(\u0040\u0040[a-z]+)?$', re.U)
搜索
word, freq = line.split(' ')[:2]
改成
word, freq = line.split('\u0040\u0040')[:2]
搜索
freq = freq.strip(' ')
tag = tag.strip(' ')
改成
freq = freq.strip('@@')
tag = tag.strip('@@')

下面是posseg的init.py 文件
搜索
re_han_detail = re.compile("([\u4E00-\u9FD5]+)")
修改为
re_han_detail = re.compile("(.+)")
搜索
re_han_internal = re.compile("([\u4E00-\u9FD5a-zA-Z0-9+#&\._]+)")
修改为
re_han_internal = re.compile("(.+)")
搜索
word, _, tag = line.split(" ")
修改为
word, _, tag = line.split("\u0040\u0040")
```
4.对于标题中含有多个名字的情况。。。。。真的是很难通过算法自动识别出谁是真的摄影师

已完成的：对图片编号、一级标题、时间、地点的处理（包括对国外摄影师姓名的处理、对各种格式的时间的处理以及对）  
   - 日期问题：用了多种正则匹配方式，已几乎可以提取出所有的日期格式，同时存在一些没有月份和日的数据采用置空处理  
   - 地点和事件：多数仍需要在自定义字典中进行添加  
   - 摄影师：完成了对外国摄影师姓名的提取，包括英文名和中文名并将之添加至词典中

未完成的：如何在多个人名中识别出摄影师的名字；关键事件词的优化；程序运行缓慢的问题（包括加载自定义字典导致的速度慢以及程序处理逻辑上的问题）




带提示的输出：
```
----------------------------------------------------------------------------------------------------
开始进行时间提取：Z:\yuexun\2016图编外拍\李晓尹\李晓尹中国2016年7月27日北京绝艺花丝镶嵌何洁\李晓尹中国2016年7月27日北京绝艺天坛
时间：2016年7月27
提取出的年月日：2016  7  27
所有的摄影师:['何洁', '2016年', '李晓尹']
摄影师:李晓尹
该照片拍摄于：['中国']
通过cut获取的关键字：['北京绝艺', '天坛', '中国', '何洁', '花丝镶嵌']
----------------------------------------------------------------------------------------------------
```
最终的输出结果：  
```
    图片编号                   一级标题	年            月              日           摄影师	拍摄地       关键词（事件名、地点） 
00000236.jpg	2016图编外拍	2016	04	09	桑浥	[]	['平谷', '桃花节']	
00000237.jpg	2016图编外拍	2016	04	09	桑浥	[]	['平谷', '桃花节']	
00000123.JPG	2016图编外拍	2016	03	23	桑浥	['奇石馆']	['奇石馆']	
00000058.jpg	2016图编外拍	2016	6	12	李晓尹	['中国']	['合影', '央美博士李玉川', '母女', '中国']	
00000059.jpg	2016图编外拍	2016	6	12	李晓尹	['中国']	['央美博士李玉川', '中国']	
00000084.jpg	2016图编外拍	2016	7	16	李晓尹	['中国', '香山']	['激情', '中关村科教惠民体验游', '中国', '香山', '帐篷']	
00000085.JPG	2016图编外拍	2016	7	27	李晓尹	['中国']	['北京绝艺', '产品', '中国', '何洁', '花丝镶嵌']	```