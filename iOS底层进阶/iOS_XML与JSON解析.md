XML与JSON简介

XML

1.  可扩展标记语言
2.  用于标记电子文件使其具有结构性的标记语言，可以用来标记数据、定义数据类型，是一种允许用户对自己的标记语言进行定义的源语言
3.  易读性高，编码手写难度小，数据量大

NSXMLParser解析方法

我们所用到的NSXMLParser是采用SAX方法解析

SAX（Simple API for XML）

*   只能读，不能修改，只能顺序访问，适合解析大型XML，解析速度快
*   常应用于处理大量数据的XML，实现异构系统的数据访问，实现跨平台
*   从文档的开始通过每一节点移动，定位一个特定的节点

DOM（Document Object Model）

*   不仅能读，还能修改，而且能够实现随机访问，缺点是解析速度慢，适合解析小型文档
*   一般应用与小型的配置XML，方便操作
*   为载入到内存的文档节点建立类型描述，呈现可横向移动、潜在巨大的树型结构
*   在内存中生成节点树操作代价昂贵

xmlParser解析过程

NSXMLParser解析过程

1.创建NSXMLParser实例，并传入从服务器接收的XML数据

2.定义解析器代理

3.解析器解析

4.通过解析代理方法完成XML数据的解析

使用XML解析文档时使用协议<NSXMLParserDelegate>，实现它的代理方法

// 1\. 开始解析某个元素，会遍历整个XML，识别元素节点名称

- (void)parser:didStartElement:namespaceURI:qualifiedName:attributes:

// 2\. 文本节点，得到文本节点里存储的信息数据，对于大数据可能会接收多次！为了节约内存开销

- (void)parser:foundCharacters:

// 3\. 结束某个节点，存储从parser:foundCharacters:方法中获取到的信息

- (void)parser:didEndElement:namespaceURI:qualifiedName:

 注意：在解析过程中，上述三个方法会不停的重复执行，直到遍历完成为止

 // 4\. 开始解析XML文档

- (void)parserDidStartDocument:

// 5\. 解析XML文档结束

- (void)parserDidEndDocument:

// 6\. 解析出错

- (void)parser:parseErrorOccurred:

接下来通过案例来详细分析XML的解析
```
#import "ViewController.h"

#import "Student.h"

@interface ViewController ()<NSXMLParserDelegate>

//student数组

@property(strong,nonatomic)NSMutableArray *students;

@property(strong,nonatomic)Student *currentStudent;

//当前的元素名

@property(strong,nonatomic)NSString *elementName;

//元素值

@property(strong,nonatomic)NSMutableString *elementValue;

@end

@implementation ViewController

//XMLParse解析过程`

-(``void``)XMLParse

{
//1.从网络上获取xml文件数据`

NSData *xmlData = [NSData dataWithContentsOfURL:[NSURL URLWithString:@``"[http://127.0.0.1/userManager/student.xml](http://127.0.0.1/userManager/student.xml)"``]];
//2.创建XML解析对象`

NSXMLParser *parser = [[NSXMLParser alloc]initWithData:xmlData];

//3.设置代理

parser.delegate = self;

//4.开始解析

[parser parse];

}

- (``void``)viewDidLoad {

[``super` `viewDidLoad];

//调用解析方法

[self XMLParse];

}

#pragma mark - NSXMLParser代理方法

//开始解析，解析的准备工作

-(``void``)parserDidStartDocument:(NSXMLParser *)parser

{

self.students = [NSMutableArray array];

}

//解析XML结束，输出文档的内容

-(``void``)parserDidEndDocument:(NSXMLParser *)parser

{

NSLog(@``"%@"``,self.students);

}

//开始解析某个元素，会遍历整个XML，识别元素结点名称

-(``void``)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary *)attributeDict

 {

if ([elementName isEqualToString:@``"student"``])

{

//创建新的对象

self.currentStudent = [[Student alloc]init];

}

else if([elementName isEqualToString:@``"name"``]||

[elementName isEqualToString:@``"age"``]||

[elementName isEqualToString:@``"sex"``])

{

self.elementValue = [[NSMutableString alloc]init];

}

}

//文本节点，得到文本节点里存储的信息数据，对于大数据可能会接收多次，

-(``void``)parser:(NSXMLParser *)parser foundCharacters:(NSString *)string

{

//将找到的字符，跟已取的字符进行拼接

[self.elementValue appendString:string];

}

//结束某个节点，存储获取到的信息

-(``void``)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName

{

if ([elementName isEqualToString:@``"name"``])

{

//将读取到的元素值，赋值给对应的属性名

self.currentStudent.name = self.elementValue;

}

else if ([elementName isEqualToString:@``"age"``])

{

self.currentStudent.age = [self.elementValue integerValue];

}

else if ([elementName isEqualToString:@``"sex"``])

{

self.currentStudent.sex = [self.elementValue characterAtIndex:``0``];

}

else if ([elementName isEqualToString:@``"student"``])

{

if (self.currentStudent)

{

[self.students addObject:self.currentStudent];

}

}

}

//差错解析

-(``void``)parser:(NSXMLParser *)parser parseErrorOccurred:(NSError *)parseError

{

NSLog(@``"Error reason:%@"``,parseError);

}

@end

```
下面这个案例就是实现citier.xml的解析
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "">
<plist version="1.0">
<dict>
    <key>provinces</key>
    <array>
        <string>安徽</string>
        <string>澳门</string>
        <string>北京</string>
        <string>重庆</string>
        <string>福建</string>
        <string>甘肃</string>
        <string>广东</string>
        <string>广西</string>
        <string>贵州</string>
        <string>海南</string>
        <string>河北</string>
        <string>河南</string>
        <string>黑龙江</string>
        <string>湖北</string>
        <string>湖南</string>
        <string>吉林</string>
        <string>江苏</string>
        <string>江西</string>
        <string>辽宁</string>
        <string>内蒙古</string>
        <string>宁夏</string>
        <string>青海</string>
        <string>山东</string>
        <string>山西</string>
        <string>陕西</string>
        <string>上海</string>
        <string>四川</string>
        <string>台湾</string>
        <string>天津</string>
        <string>西藏</string>
        <string>香港</string>
        <string>新疆</string>
        <string>云南</string>
        <string>浙江</string>
    </array>
    <key>cities</key>
    <dict>
        <key>北京</key>
        <array>
            <string>东城区</string>
            <string>西城区</string>
            <string>崇文区</string>
            <string>宣武区</string>
            <string>海淀区</string>
            <string>朝阳区</string>
            <string>丰台区</string>
            <string>石景山区</string>
            <string>通州区</string>
            <string>顺义区</string>
            <string>房山区</string>
            <string>大兴区</string>
            <string>昌平区</string>
            <string>怀柔区</string>
            <string>平谷区</string>
            <string>门头沟区</string>
            <string>密云县</string>
            <string>延庆县</string>
        </array>
        <key>重庆</key>
        <array>
            <string>重庆</string>
        </array>
        <key>上海</key>
        <array>
            <string>上海</string>
        </array>
        <key>天津</key>
        <array>
            <string>天津</string>
        </array>
        <key>安徽</key>
        <array>
            <string>合肥</string>
            <string>安庆</string>
            <string>蚌埠</string>
            <string>亳州</string>
            <string>巢湖</string>
            <string>池州</string>
            <string>滁州</string>
            <string>阜阳</string>
            <string>淮北</string>
            <string>淮南</string>
            <string>黄山</string>
            <string>黄山景区</string>
            <string>九华山景区</string>
            <string>六安</string>
            <string>马鞍山</string>
            <string>青阳</string>
            <string>宿州</string>
            <string>铜陵</string>
            <string>芜湖</string>
            <string>宣城</string>
        </array>
        <key>福建</key>
        <array>
            <string>福州</string>
            <string>龙岩</string>
            <string>南平</string>
            <string>宁德</string>
            <string>莆田</string>
            <string>泉州</string>
            <string>三明</string>
            <string>厦门</string>
            <string>永安</string>
            <string>漳州</string>
        </array>
        <key>广东</key>
        <array>
            <string>广州</string>
            <string>潮州</string>
            <string>从化</string>
            <string>东莞</string>
            <string>佛山</string>
            <string>河源</string>
            <string>鹤山</string>
            <string>化州</string>
            <string>惠州</string>
            <string>江门</string>
            <string>揭阳</string>
            <string>茂名</string>
            <string>梅州</string>
            <string>清远</string>
            <string>汕头</string>
            <string>汕尾</string>
            <string>韶关</string>
            <string>深圳</string>
            <string>阳江</string>
            <string>云浮</string>
            <string>湛江</string>
            <string>肇庆</string>
            <string>中山</string>
            <string>珠海</string>
        </array>
        <key>甘肃</key>
        <array>
            <string>兰州</string>
            <string>白银</string>
            <string>定西</string>
            <string>甘南</string>
            <string>嘉峪关</string>
            <string>酒泉</string>
            <string>临夏</string>
            <string>陇南</string>
            <string>平凉</string>
            <string>庆阳</string>
            <string>天水</string>
            <string>武威</string>
            <string>张掖</string>
        </array>
        <key>广西</key>
        <array>
            <string>南宁</string>
            <string>百色</string>
            <string>北海</string>
            <string>北流</string>
            <string>崇左</string>
            <string>防城港</string>
            <string>贵港</string>
            <string>桂林</string>
            <string>桂平</string>
            <string>河池</string>
            <string>贺州</string>
            <string>来宾</string>
            <string>柳州</string>
            <string>钦州</string>
            <string>梧州</string>
            <string>宜州</string>
            <string>玉林</string>
        </array>
        <key>贵州</key>
        <array>
            <string>贵阳</string>
            <string>安顺</string>
            <string>毕节</string>
            <string>都匀</string>
            <string>凯里</string>
            <string>六盘水</string>
            <string>铜仁</string>
            <string>兴义</string>
            <string>遵义</string>
        </array>
        <key>河北</key>
        <array>
            <string>石家庄</string>
            <string>保定</string>
            <string>泊头</string>
            <string>沧州</string>
            <string>承德</string>
            <string>邯郸</string>
            <string>河间</string>
            <string>衡水</string>
            <string>廊坊</string>
            <string>秦皇岛</string>
            <string>任丘</string>
            <string>唐山</string>
            <string>邢台</string>
            <string>张家口</string>
        </array>
        <key>河南</key>
        <array>
            <string>郑州</string>
            <string>安阳</string>
            <string>鹤壁</string>
            <string>济源</string>
            <string>焦作</string>
            <string>开封</string>
            <string>洛阳</string>
            <string>漯河</string>
            <string>南阳</string>
            <string>平顶山</string>
            <string>濮阳</string>
            <string>三门峡</string>
            <string>商丘</string>
            <string>新乡</string>
            <string>信阳</string>
            <string>许昌</string>
            <string>周口</string>
            <string>驻马店</string>
        </array>
        <key>黑龙江</key>
        <array>
            <string>哈尔滨</string>
            <string>大庆</string>
            <string>大兴安岭</string>
            <string>鹤岗</string>
            <string>黑河</string>
            <string>虎林</string>
            <string>鸡西</string>
            <string>佳木斯</string>
            <string>密山</string>
            <string>牡丹江</string>
            <string>宁安</string>
            <string>七台河</string>
            <string>齐齐哈尔</string>
            <string>双鸭山</string>
            <string>绥化</string>
            <string>五常</string>
            <string>伊春</string>
        </array>
        <key>湖北</key>
        <array>
            <string>武汉</string>
            <string>鄂州</string>
            <string>恩施</string>
            <string>黄冈</string>
            <string>黄石</string>
            <string>荆门</string>
            <string>荆州</string>
            <string>潜江</string>
            <string>十堰</string>
            <string>随州</string>
            <string>天门</string>
            <string>仙桃</string>
            <string>咸宁</string>
            <string>襄樊</string>
            <string>孝感</string>
            <string>宜昌</string>
        </array>
        <key>湖南</key>
        <array>
            <string>长沙</string>
            <string>常德</string>
            <string>郴州</string>
            <string>衡阳</string>
            <string>怀化</string>
            <string>吉首</string>
            <string>耒阳</string>
            <string>冷水江</string>
            <string>娄底</string>
            <string>韶山</string>
            <string>邵阳</string>
            <string>湘潭</string>
            <string>湘乡</string>
            <string>益阳</string>
            <string>永州</string>
            <string>岳阳</string>
            <string>张家界</string>
            <string>株州</string>
        </array>
        <key>吉林</key>
        <array>
            <string>长春</string>
            <string>白城</string>
            <string>白山</string>
            <string>珲春</string>
            <string>吉林</string>
            <string>辽源</string>
            <string>龙井</string>
            <string>舒兰</string>
            <string>四平</string>
            <string>松原</string>
            <string>通化</string>
            <string>延边</string>
        </array>
        <key>江苏</key>
        <array>
            <string>南京</string>
            <string>常州</string>
            <string>高邮</string>
            <string>淮安</string>
            <string>连云港</string>
            <string>南通</string>
            <string>苏州</string>
            <string>宿迁</string>
            <string>太仓</string>
            <string>泰州</string>
            <string>无锡</string>
            <string>新沂</string>
            <string>徐州</string>
            <string>盐城</string>
            <string>扬州</string>
            <string>镇江</string>
        </array>
        <key>江西</key>
        <array>
            <string>南昌</string>
            <string>抚州</string>
            <string>赣州</string>
            <string>吉安</string>
            <string>景德镇</string>
            <string>九江</string>
            <string>萍乡</string>
            <string>上饶</string>
            <string>新余</string>
            <string>宜春</string>
            <string>鹰潭</string>
        </array>
        <key>辽宁</key>
        <array>
            <string>沈阳</string>
            <string>鞍山</string>
            <string>本溪</string>
            <string>朝阳</string>
            <string>大连</string>
            <string>丹东</string>
            <string>抚顺</string>
            <string>阜新</string>
            <string>葫芦岛</string>
            <string>锦州</string>
            <string>辽阳</string>
            <string>盘锦</string>
            <string>铁岭</string>
            <string>营口</string>
        </array>
        <key>内蒙古</key>
        <array>
            <string>呼和浩特</string>
            <string>阿拉善盟</string>
            <string>巴彦淖尔盟</string>
            <string>包头</string>
            <string>赤峰</string>
            <string>鄂尔多斯</string>
            <string>呼伦贝尔</string>
            <string>通辽</string>
            <string>乌海</string>
            <string>乌兰察布盟</string>
            <string>锡林郭勒盟</string>
            <string>兴安盟</string>
        </array>
        <key>宁夏</key>
        <array>
            <string>银川</string>
            <string>固原</string>
            <string>石嘴山</string>
            <string>吴忠</string>
            <string>中卫</string>
        </array>
        <key>青海</key>
        <array>
            <string>西宁</string>
            <string>果洛</string>
            <string>海北</string>
            <string>海东</string>
            <string>海南</string>
            <string>海西</string>
            <string>黄南</string>
            <string>玉树</string>
        </array>
        <key>四川</key>
        <array>
            <string>成都</string>
            <string>阿坝</string>
            <string>巴中</string>
            <string>崇州</string>
            <string>达州</string>
            <string>大邑</string>
            <string>德阳</string>
            <string>都江堰</string>
            <string>峨眉山</string>
            <string>甘孜</string>
            <string>广安</string>
            <string>广元</string>
            <string>江油</string>
            <string>金堂</string>
            <string>乐山</string>
            <string>泸州</string>
            <string>眉山</string>
            <string>绵阳</string>
            <string>内江</string>
            <string>南充</string>
            <string>攀枝花</string>
            <string>遂宁</string>
            <string>西昌</string>
            <string>雅安</string>
            <string>宜宾</string>
            <string>资阳</string>
            <string>自贡</string>
        </array>
        <key>山东</key>
        <array>
            <string>济南</string>
            <string>滨州</string>
            <string>德州</string>
            <string>东营</string>
            <string>肥城</string>
            <string>海阳</string>
            <string>菏泽</string>
            <string>济宁</string>
            <string>莱芜</string>
            <string>莱阳</string>
            <string>聊城</string>
            <string>临沂</string>
            <string>平度</string>
            <string>青岛</string>
            <string>青州</string>
            <string>日照</string>
            <string>泰安</string>
            <string>威海</string>
            <string>潍坊</string>
            <string>烟台</string>
            <string>枣庄</string>
            <string>章丘</string>
            <string>淄博</string>
        </array>
        <key>陕西</key>
        <array>
            <string>西安</string>
            <string>安康</string>
            <string>宝鸡</string>
            <string>汉中</string>
            <string>商洛</string>
            <string>铜川</string>
            <string>渭南</string>
            <string>咸阳</string>
            <string>兴平</string>
            <string>延安</string>
            <string>榆林</string>
        </array>
        <key>山西</key>
        <array>
            <string>太原</string>
            <string>长治</string>
            <string>大同</string>
            <string>晋城</string>
            <string>晋中</string>
            <string>临汾</string>
            <string>吕梁</string>
            <string>朔州</string>
            <string>忻州</string>
            <string>阳泉</string>
            <string>运城</string>
        </array>
        <key>新疆</key>
        <array>
            <string>乌鲁木齐</string>
            <string>阿克苏</string>
            <string>阿拉尔</string>
            <string>阿图什</string>
            <string>博乐</string>
            <string>昌吉</string>
            <string>哈密</string>
            <string>和田</string>
            <string>喀什</string>
            <string>克拉玛依</string>
            <string>库尔勒</string>
            <string>石河子</string>
            <string>图木舒克</string>
            <string>吐鲁番</string>
            <string>五家渠</string>
            <string>伊宁</string>
        </array>
        <key>西藏</key>
        <array>
            <string>拉萨</string>
            <string>阿里</string>
            <string>昌都</string>
            <string>林芝</string>
            <string>那曲</string>
            <string>日喀则</string>
            <string>山南</string>
        </array>
        <key>云南</key>
        <array>
            <string>昆明</string>
            <string>保山</string>
            <string>楚雄</string>
            <string>大理</string>
            <string>德宏</string>
            <string>迪庆</string>
            <string>个旧</string>
            <string>丽江</string>
            <string>临沧</string>
            <string>怒江</string>
            <string>曲靖</string>
            <string>思茅</string>
            <string>文山</string>
            <string>西双版纳</string>
            <string>玉溪</string>
            <string>昭通</string>
        </array>
        <key>浙江</key>
        <array>
            <string>杭州</string>
            <string>北仑</string>
            <string>慈溪</string>
            <string>奉化</string>
            <string>湖州</string>
            <string>嘉兴</string>
            <string>金华</string>
            <string>丽水</string>
            <string>临海</string>
            <string>宁波</string>
            <string>宁海</string>
            <string>衢州</string>
            <string>三门</string>
            <string>绍兴</string>
            <string>台州</string>
            <string>天台</string>
            <string>温岭</string>
            <string>温州</string>
            <string>仙居</string>
            <string>象山</string>
            <string>义乌</string>
            <string>余姚</string>
            <string>舟山</string>
        </array>
        <key>澳门</key>
        <array>
            <string>澳门</string>
        </array>
        <key>台湾</key>
        <array>
            <string>台北</string>
            <string>高雄</string>
            <string>台南</string>
            <string>台中</string>
        </array>
        <key>香港</key>
        <array>
            <string>香港</string>
        </array>
        <key>海南</key>
        <array>
            <string>海口</string>
            <string>儋州</string>
            <string>东方</string>
            <string>琼海</string>
            <string>三亚</string>
            <string>万宁</string>
            <string>文昌</string>
            <string>五指山</string>
        </array>
    </dict>
</dict>
</plist>
```
cities.xml
实现代码如下：
```
 #import "ViewController.h"
 
 @interface ViewController ()<NSXMLParserDelegate>
 @property(strong,nonatomic)NSMutableDictionary *dic;
 @property(strong,nonatomic)NSMutableArray *array;
 @property(strong,nonatomic)NSMutableDictionary *cities;
  @property(strong,nonatomic)NSString *keyName;
  @property(strong,nonatomic)NSMutableString *elementValue;

 @end

 @implementation ViewController

 - (void)viewDidLoad {
     [super viewDidLoad];
     [self XMLParser];
 }

 -(void)XMLParser
 {
     //1.获取网络数据
     NSData *xmlData = [NSData dataWithContentsOfURL:[NSURL URLWithString:@""]];
     //2.创建解析对象
     NSXMLParser *parser = [[NSXMLParser alloc]initWithData:xmlData];
     //3.设置代理
     parser.delegate = self;
     //4.开始解析
     [parser parse];
 }
 #pragma mark - NSXMLParser代理方法
 //文档开始
 -(void)parserDidStartDocument:(NSXMLParser *)parser
 {
     self.dic = [NSMutableDictionary dictionary];
 }
 //文档结束
 -(void)parserDidEndDocument:(NSXMLParser *)parser
 {
    [self.dic setObject:self.cities forKey:@"cities"];
     NSLog(@"%@",self.dic);
 }
 //元素开始
 -(void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary *)attributeDict
 {
     if ([elementName isEqualToString:@"array"])
    {
         self.array = [NSMutableArray array];
     }
     else if ([elementName isEqualToString:@"dict"])
     {
         if ([self.keyName hasPrefix:@"cities"])
         {
             self.cities = [NSMutableDictionary dictionary];
         }
     }
     else if ([elementName isEqualToString:@"string"]||
              [elementName isEqualToString:@"key"])
     {
         self.elementValue = [[NSMutableString alloc]init];
     }
 }
 //元素结束
 -(void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName
 {
     if ([elementName isEqualToString:@"array"])
     {
         if ([self.keyName hasPrefix:@"provinces"])
         {
            [self.dic setObject:self.array forKey:self.keyName];
        }
         else
         {
             [self.cities setObject:self.array forKey:self.keyName];
         }
     }
     else if ([elementName isEqualToString:@"key"])
     {
         self.keyName = self.elementValue;
     }
     else if ([elementName isEqualToString:@"string"])
     {
         [self.array addObject:self.elementValue];
    }
    
 }
//找字符
 -(void)parser:(NSXMLParser *)parser foundCharacters:(NSString *)string
 {
     [self.elementValue appendString:string];
 }
 //出错解析
 -(void)parser:(NSXMLParser *)parser parseErrorOccurred:(NSError *)parseError
 {
    NSLog(@"error reason:%@",parseError);
 }
@end
```
JSON
作为一种轻量级的数据交换格式，正在逐步取代XML，成为网络数据的通用格式
基于JavaScript的一个子集
易读性略差，编码手写难度大，数据量小
JSON格式取代了XML给网络传输带来了很大的便利，但是却没有了XML的一目了然，尤其是JSON数据很长的时候，我们会陷入繁琐复杂的数据节点查找中
作为一种轻量级的数据交换格式，JSON正在逐步取代XML，成为网络数据的通用格式。废话不多说，直接上代码。
案例1:使用NSDictionary创建JSON文件
```
-(void)dataFromJSONObject
 {
     //从网络数据生成字典
     NSDictionary *dic = [NSDictionary dictionaryWithContentsOfURL:[NSURL URLWithString:@""]];
     NSError *error = nil;
    //将字典转换成JSON数据
     //    NSJSONWritingPrettyPrinted：的意思是将生成的json数据格式化输出，这样可读性高，不设置则输出的json字符串就是一整行。
     NSData *data = [NSJSONSerialization dataWithJSONObject:dic options:NSJSONWritingPrettyPrinted error:&error];
     if (!error)
     {
         //拼接路径
        NSString *document = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
         NSLog(@"%@",document);
         NSString *jsonPath = [document stringByAppendingPathComponent:@"cities.json"];
         //将数据写入文件中
         [[NSFileManager defaultManager]createFileAtPath:jsonPath contents:data attributes:nil];
     }
 }
```
案例2：读取已有的JSON文件内容
```
-(void)JSONObjectFromData
 {
     NSURL *url = [NSURL URLWithString:@""];
     NSData *data = [NSData dataWithContentsOfURL:url];
     NSError *error = nil;
     //读取文件内容
     id obj = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingAllowFragments error:&error];
     if (!error)
     {
        //判断对象的类型
         if ([obj isKindOfClass:[NSDictionary class]])
         {
             NSDictionary *dic = obj;
             NSLog(@"dic");
             NSArray *provinces = [dic objectForKey:@"provinces"];
             [provinces enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop)
             {
                 NSLog(@"%@",obj);
            }];
         }
        else if ([obj isKindOfClass:[NSArray class]])
         {
            NSArray *array = obj;
             NSLog(@"array");
         }
     }
 }
```
>最后有什么疑惑问题这有个iOS交流群：[642363427](https://links.jianshu.com/go?to=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2Flinks.jianshu.com%2Fgo%253Fto%253Dhttps%25253A%25252F%25252Flink.zhihu.com%25252F%25253Ftarget%25253Dhttps%2525253A%25252F%25252Fjq.qq.com%25252F%2525253F_wv%2525253D1027%25252526k%2525253D15vUEWzp)有一个共同的圈子很重要，结识人脉！里面都是iOS开发，全栈发展，欢迎入驻，共同进步！（群内会免费提供一些群主收藏的免费学习书籍资料以及整理好的几百道面试题和答案文档！）
