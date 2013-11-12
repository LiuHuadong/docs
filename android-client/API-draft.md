# API draft
------------

## 模型

```java

// 代表K-V形式选项
class Options extends LinkedHashMap<String, Object> {
}

// 描述真实货币
class RealMoney {
    String symbol; // 货币符号
    double amount; // 金额
}

// 花费
class Cost {
    // 常量
    static final int PRIMARY_CURRENCY = 1;
    static final int SECOND_CURRENCY = 2;
    static final int REAL_MONEY = 3;
    
    int type;               // PRIMARY_CURRENCY or SECOND_CURRENCY or REAL_MONDY
    Object cost;            // 根据 type 不同，所代表内容为Long或者RealMoney的值
}

// 描述文本
class Text {
    String text; // 文本内容
}

// 描述应用程序版本条件
class AppCondition {
    Integer minVersion;     // 最小版本
    Integer maxVersion;     // 最大版本
}

// 客户端应用上下文
class ConsumerContext {
    String app;             // App ID
    String device;          // Device ID
} 


// 货物
class Goods {
    // 常量
    static final int FREE = 1; // 免费
    static final int PAID = 2; // 付费
    static final int ANON_FREE = 3; // 对未登录用户免费

    static final int CT_NONE = 0;  // 没有内容
    static final int CT_URL = 1;   // 下载链接
    static final int CT_TEXT = 2;  // 文本内容

    // 商品属性
    String id;              // ID
    long version;           // 版本，使用时间戳表示
    DateTime createdAt;     // 创建时间
    DateTime updatedAt;     // 更新内容时间
    String[] tags;          // 所拥有的ｔａｇｓ
    String[] previewImageURLs;  // 预览图片列表
    Text name;              // 名称
    Text description;       // 描述
    String appData;         // 用户自定义文本信息
    int paidType;           // FREE or PAID or ANON_FREE
    long primaryCurrency;   // 主货币售价    \
    long secondCurrency;    // 次货币售价    --不能组合使用３种货币  
    RealMoney realMoney;    // 真实货币售价   /
    boolean consumable;     // 货物是否可被消费掉
    int limitPerUser;       // 每用户拥有此货物数量上限
    AppCondition appCondition; // 对客户端Ａpp的需求条件
    int contentType;                // CT_NONE or CT_URL or CT_TEXT
    String content;                 // 内容，如果type=URL，则为URL;如果type=TEXT，则为文本内容 
    List<Goods> subGoods;           // 子货物列表

    // 其他动态属性
    long salesCount;        // 销量
    // ...   

    // 带有数量的货物
    class WithCount extends Goods {
        int count;          // 货物的数量
    }
}

// 货物分类
class Category {
    String id;              // ID
    Text name;              // 名称
    Text description;       // 描述
}


class History {
    String id;              // ID
    String createdAt;       // 时间
    Goods.WithCount goods;  // 货物及其数量
}

// 购买收据
class Receipt extends History {
    Cost cost;              // 购买所付出的花费
}

// 消费记录
class Consumption extends History {
}

// 个人资产
class Assets {
    long primaryCurrency;           // 主货币
    long secondCurrency;            // 次货币
    List<Goods.WithCount> goods;    // 当前拥有的货物及数量
}

// 用户信息
class User {
    String uid;             // 用户ID
    String human;           // 可读的UID，例如email
    String appData;         // 程序为此附加的文本信息
    String ticket;          // 登录ticket
}

```

------------

## APIs

```java

interface IdentifyUserCallback {
    void done(User user, Exception error);
}

interface CategoriesCallback {
    void done(List<Categories> categories, Exception error);
}

interface GoodsListCallback {
    void done(List<Goods> goods, Exception error);
}

interface BuyCallback {
    void done(Receipt receipt, Exception error);
}

interface ConsumeCallback {
    void done(Consumption consumption, Exception error);
}

interface AssetsCallback {
    void done(User user, Assets assets, Exception error);
}

interface HistoriesCallback {
    void done(List<History> histories, Exceptione error);
}

class VGClient {
    
    // 构造函数
    public VGClient(
            ConsumerContext ctx,    // 此应用上下文 
            Options opts            // 其他选项
        );          
    
    // 获取当前应用上下文
    ConsumerContext getConsumerContext();
    
    

    // 标识此客户端用户分身
    void identifyUser(
            User user,              // 用户信息
            IdentifyUserCallback cb
        );

    // 获取当前用户信息
    User getCurrentUser();  

    // 是否已经标识过此客户端用户信息        
    boolean isIdentified();         

    
    // 列出此应用内货物分类列表
    void listCategories(ListCategoriesCallback cb);
    
    // 查询可货物列表
    void queryGoods(
            String category,        // 分类ID 
            String searchKey,       // 搜索关键字
            Options opts,           // 其他选项，包括分页，排序，tag等
            GoodsListCallback cb
        );

    // 购买货物
    void buyGoods(
            String goodsId,         // 购买货物ID 
            int costType,           // 使用虚拟货币还是真实货币付费
            int count,              // 购买数量
            Options opts,           // 其他选项
            BuyCallback cb
        );
    
    // 发货
    void giveGoods(
            String[] goodsId,       // 要求服务器发货的货物ID列表
            Options opts,           // 其他选项 
            GoodsListCallback cb    
        );

    // 消费货物
    void consumeGoods(
            String goodsId,         // 消费的货物ID 
            int count,              // 消费掉的数量
            Options opts,           // 其他选项
            ConsumeCallback cb
    );

    // 获取此用户当前拥有的资产
    void getAssets(AssetsCallback cb);

    // 列出购买和消费历史记录
    void listHistories(
            Options opts,           // 选项，例如时间范围 
            HistoriesCallback cb
        );
}


```