# java
概述
本文是沐金农开发推荐的编码建议。众所周知，好的代码一定是"简洁、明确、逻辑清晰并且好看的代码"。写得一手好代码有助于你的提级审核。
名词解释
原则
设计环节
无单点
会话无状态（Stateless）
可运维，可监控
可测试
可回滚，可禁用
可水平拆分
可缓存
可复制
短事务，最终一致性柔性事务
并发控制
异步处理
支持分级与降级
实现环节
代码清晰简洁，易于阅读
为了公司编码尽可能统一，公司统一推荐使用springmvc。业务逻辑后移到Service层，如果业务系统非常复杂，可以在Controller和service之间增加一层Biz层来负责组织比较复杂的业务逻辑，Controller层轻薄，Controller层职责为用户输入数据验证，逻辑分发，输出数据封装。
合理组织代码层次，根据代码职责组织代码。
排版清晰，易于阅读，代码必须格式化。建议在代码逻辑块之间留有空行。代码范例：
 // 分配trans_log_id-19位
String transLogPre = TransLogSeqUtil.construct(transInfo.getTransDate(), String.valueOf(seqDAO.getNextSeqFtTransInfo()));
transInfo.setTransLogSeq(transLogPre);

// 转换
TransInfoDO transInfoDO = TransInfoConverter.convertTransInfoToDO(transInfo);
return String.valueOf(transInfoDAO.insert(transInfoDO));
建议面向接口编程，使用Spring来管理bean的生命周期。
代码必须格式化。IDE的格式化配置：IDEA code format setting
注释规范
类注释，必须描述该类的职责，建议写上类作者，版本，日期 代码范例：
/**
 * Created by BaoJiawei on 5/4/2015.
 * desc:
 * 调用国内
 */
@Controller
@RequestMapping(value = "/api/v1/market")
@Log4j
public class MarketController extends BaseController {
    List<String> list = new ArrayList<String>();
}
方法注释，必须描述该方法的职责，对于方法的入参与出参建议加以必要的明确的注释，对于一目了然的代码逻辑，可以不写
代码范例：
   /**
 * 根据用户风测结果，计算用户所属投资类型，实际测试结果需要对比风险容忍度结果：
 * 高则风格不变，
 * 中对应稳健、成长、保守型（测试结果为积极和非常积极，变更结果为稳健型），
 * 低则不管结果如何均对应保守型。
 * 同时更新用户投资目标，对应关系为高中低—投机增值交易利润。
 * @param roboAccountId
 * @param riskTolerance
 * @param model
 * @return
 */
    private String updateInvestmentTypeByRiskTolerance(Integer roboAccountId, SurveyParameter riskTolerance, String model) {
        if (riskTolerance.getOptionId() == HIGH_RISK_TOLERANCE) {
        //不变
        } else if (riskTolerance.getOptionId() == MEDIUM_RISK_TOLERANCE) {
            if (model.equals(VERY_POSITIVE) || model.equals(POSITIVE)) {
                log.info("roboAccountId:"+roboAccountId+"change riskTolerance "+model+"-->"+"稳健型");
                model = PRUDENT;
            }
        //稳健,成长,保守
        } else if (riskTolerance.getOptionId() == LOW_RISK_TOLERANCE) {
            log.info("roboAccountId:"+roboAccountId+"change riskTolerance "+model+"-->"+"保守型");
            model = CONSERVATIVE;
            //保守型
        }
        return model;
全局变量的注释，对于全局变量，必须加以明确的注释描述。 范例代码：
/**
 * redis key - 美股交易状态
 */
public static final String REDIS_US_EXCHANGE_IS_OPEN = "REDIS_US_EXCHANGE_IS_OPEN";
/**
 * redis key - 热门美股涨幅榜
 */
public static final String REDIS_US_GLAMOUR = "REDIS_US_GLAMOUR";

/**
 * redis key - 中概股涨幅榜
 */
public static final String REDIS_US_CCS = "REDIS_US_CCS";
对于方法中的逻辑代码块，需要加明确的行级注释，便于快速了解一个方法的大概逻辑步骤。注释的位置在代码的上方，左对齐。
代码范例：
// 获取风险等级描述与风险type对照信息
List<RARiskMapping> raRiskMappings = robotAdviserService.findRiskMappingByDes(investmentType.getName());

// 通过风险type获取投资模型
RAPortfolioModel raPortfolioModel = robotAdviserService.findLastRAModelByTypes(collect);
接口注释，对于接口的职责必须有明确的注释，对于接口中的方法，需要说明方法的职责，方法的入参，方法的出参，此外，对于使用场景复杂的方法，建议加上方法的使用范例(特别是Dubbo对外接口)。 代码范例：
/**
 * XXXX信息明细分页查询
 * @param <pre>AccountLogPageQuery
 * <li>minDate : 起始业务日期(yyyyMMdd) </li>
 * <li>maxDate : 终止业务日期(yyyyMMdd) </li>
 * <li>accountNo ：账号 </li>
 * <li>
 * accountType:XXXX枚举类，
 * <li>XXXX -1 </li>
 * <li>XXXX -2 </li>
 * <li>全部 -3 </li>
 * </li>
 * <li>
 * enueCodeEnum:业务代码枚举类，
 * <li>1001 - XXXX</li>
 * <li>2012 - XXXX</li>
 * <li>2031 - XXXX</li>
 * <li>3011 - XXXX</li>
 * </li>
 * <li>companyCode :公司编码 01 ,02 ,...不能为空</li>
 * <li>pageSize: 每页条数，默认为20</li>
 * <li>currentPage: 当前页码，默认为1</li>
 * </pre>
 *
 * @return <pre>XXXXXXXtLogPageResult:账户明细信息分页查询结果对象
 * <li>success: 成功标识</li>
 * <li>code: 结果码</li>
 * <li>message: 返回信息</li>
 *
 * <li>accountLogInfos: 当前页的账户明细信息列表，未查询到结果时为空对象</li>
 * <li>totalItems: 查询到的结果总数</li>
 * <li>totalPages: 总页数</li>
 * <li>currentPage: 当前页码</li>
 * <li>itemsPerPage: 每页条数</li>
 */
    public XXXXXXXPageResult queryAccountLogByPage(AccountLogPageQuery accountLogPageQuery);
对于逻辑单元的代码块以及每个条件判断if建议加上适当的行级注释。一目了然的代码逻辑可不加。我们的目标只有一个：易读易懂易维护
命名规范
无论是类名称，变量名称，方法名称，遵循以下原则，简单明确，知名见义。建议不带数字，下划线等特殊字符（类常量除外），采用有意义的名词或者名称组合驼峰法命名。
不好的方法名：
public Object getDetailInfo(id){
     //查询数据库操作
}

正确方法名：
public Object queryDetailInfo(id){
     //查询数据库操作
}
方法名getDetailInfo没有queryDetailInfo好。get容易跟java中的getter,setter混淆，如果把这个当成getter操作，容易造成性能问题。
同样一个远程连接的url请求，也最好不要命名成getXXX，而换成fetchXXX更好。
对于类常量，推荐使用下划线分割的描述性名词来命名。
/** 查询条目限制 */
private static final Integer ACCOUNT_LOG_NUM = 10000;
其他通用规范
方法职责单一，方法命名以能够表达方法职责为宜。方法长度最好不超过70行。
接口职责单一，命名以能够表达职责为宜。
代码重复度小，可复用，圈复杂度不超过3 不推荐的代码范例：
        // 是否为有效的房型 
        boolean isSpecAviliable = true;

        // 是否无空的报价 
        boolean isPerfect = true;
        int sum = 0;
        int count = 0;
        for (int i = 0; i < hs.size(); i++) {
            if (!"0".equals(hs.get(i))) {
                isPerfect = false;
                if (specType == SpecType.LZ && i < specValue) {
                    isSpecAviliable = false;
                    break;
                }
            } else {
                sum = sum + rp.get(i);
                count++;
            }
        }
        if (!isSpecAviliable) {
            commissionHotel.removeCommissionRoom(commissionRoom);
            continue;
        } else {
            SpecRoom specRoom = availablePhysicalMap.get(commissionRoom.getPid());
            if (specRoom == null) {
                specRoom = new SpecRoom(commissionRoom.getId(), specValue, (float) sum / (float) count,
                        isPerfect, specType, commissionRoom);
                availablePhysicalMap.put(commissionRoom.getPid(), specRoom);
            } else {
                if (isNeedRemove(specType, specValue, isPerfect, (float) sum, (float) count, specRoom)) {
                    commissionHotel.removeCommissionRoom(specRoom.getCommissionRoom());
                    availablePhysicalMap.put(commissionRoom.getPid(), new SpecRoom(commissionRoom.getId(),
                            specValue, (float) sum / (float) count, isPerfect, specType, commissionRoom));
                } else {
                    commissionHotel.removeCommissionRoom(commissionRoom);
                }
            }
        }
推荐的代码范例：
    public CommissionRoom getRoomForAPhysicalRoom(List<CommissionRoom> roomsOfAPhysicalRoom, Map<Integer, CommissionRoomVO> productRoom) {
        removeRoomsWithoutPrice(roomsOfAPhysicalRoom);
        removeRoomsHigherAvgPrice(roomsOfAPhysicalRoom);
        List<CommissionRoom> lzRooms = filterLZRooms(roomsOfAPhysicalRoom, productRoom);
        if (lzRooms.size() > 0) {
            return getRoomForMinSpecValue(lzRooms, productRoom);
        } else {
            return getRoomForMinSpecValue(roomsOfAPhysicalRoom, productRoom);
        }
    }

    private void removeRoomsWithoutPrice(List<CommissionRoom> roomsOfAPhysicalRoom) {
        Iterator<CommissionRoom> iterator = roomsOfAPhysicalRoom.iterator();
        while (iterator.hasNext()) {
            CommissionRoom room = iterator.next();
            if (hasWithoutPriceDay(room)) {
                iterator.remove();
            }
        }
    }

    private boolean hasWithoutPriceDay(CommissionRoom room) {
        for (String roomStatus : room.getHsList()) {
            if (isFull(roomStatus)) {
                return true;
            }
        }
        return false;
    }

    private void removeRoomsHigherAvgPrice(List<CommissionRoom> roomsOfAPhysicalRoom) {
        if (CollectionUtils.isEmpty(roomsOfAPhysicalRoom)) return;
        float minAvgPrice = computeMinAvgPrice(roomsOfAPhysicalRoom);
        Iterator<CommissionRoom> iterator = roomsOfAPhysicalRoom.iterator();
        while (iterator.hasNext()) {
            CommissionRoom room = iterator.next();
            float avgPrice = computeAvgPrice(room);
            if (avgPrice > minAvgPrice) {
                iterator.remove();
            }
        }
    }

    private float computeMinAvgPrice(List<CommissionRoom> roomsOfAPhysicalRoom) {
        float minAvgPrice = computeAvgPrice(roomsOfAPhysicalRoom.get(0));
        for (CommissionRoom room : roomsOfAPhysicalRoom) {
            float avgPrice = computeAvgPrice(room);
            if (avgPrice < minAvgPrice) {
                minAvgPrice = avgPrice;
            }
        }
        return minAvgPrice;
    }

    private List<CommissionRoom> filterLZRooms(List<CommissionRoom> roomsOfAPhysicalRoom, Map<Integer, CommissionRoomVO> productRoomMap) {
        List<CommissionRoom> result = new ArrayList<CommissionRoom>();
        for (CommissionRoom commissionRoom : roomsOfAPhysicalRoom) {
            CommissionRoomVO productRoom = productRoomMap.get(commissionRoom.getId());
            if (productRoom.getSpecType() == SpecType.LZ) {
                result.add(commissionRoom);
            }
        }
        return result;
    }

    private CommissionRoom getRoomForMinSpecValue(List<CommissionRoom> rooms, Map<Integer, CommissionRoomVO> productRoomMap) {
        if (rooms.size() == 1) return rooms.get(0);
        Integer minSpecValue = productRoomMap.get(rooms.get(0).getId()).getSpecValue();
        CommissionRoom result = null;
        for (CommissionRoom room : rooms) {
            CommissionRoomVO productRoom = productRoomMap.get(room.getId());
            int specValue = productRoom.getSpecValue();
            if (specValue <= minSpecValue) {
                result = room;
                minSpecValue = specValue;
            }
        }
        return result;
    }

    private float computeAvgPrice(CommissionRoom room) {
        Integer sumPrice = 0;
        for (Integer price : room.getPriceList()) {
            sumPrice += price;
        }
        return sumPrice / room.getPriceList().size();
    }

    private boolean isFull(String roomStatus) {
        //这里不符合规范，不用用魔术字符串。 
        //可以以修改成 return !roomStatus.equals(NOTFULL) 
        return !roomStatus.equals("0");
    }
请注意，上面两段代码是同一个方法。
归集系统常量并加以注释，禁止出现魔术字符 推荐的代码范例
public enum XXXResultCode {

   /** 处理成功 */
   SUCCESS("SUCCESS", "处理成功"),

   /** 未知异常 */
   UN_KNOWN_EXCEPTION("UN_KNOWN_EXCEPTION", "未知异常"),

   /** 系统错误 */
   SYSTEM_ERROR("SYSTEM_ERROR", "系统错误"),

   /** 产品传入参数非法 */
   PRODUCT_INPUT_PARAMETER_ILLEGAL("PRODUCT_INPUT_PARAMETER_ILLEGAL", "产品传入参数非法"),
   .............
   .............
}
错误的写法：
if(specType!=null){
    if(!"0".equals(hs.get(0))){//无法理解"0"的含义
        commissionHotel.removeCommissionRoom(commissionRoom);
        continue;
    }
}

if(commissionType==0){//无法理解0的意思
    base = basePrice.multiply(new BigDecimal(baseCommissionValue)).multiply(percent).divide(new BigDecimal(10000)); //取平日价
    if(weekendCommissionValue==null||weekendPrice==null){
        weekEnd=base;
    }else{
        weekEnd=weekendPrice.multiply(new BigDecimal(weekendCommissionValue)).multiply(percent).divide(new BigDecimal(10000));//取周末价
    }
}
if(commissionType==1){//无法理解1的意思
    base = new BigDecimal(baseCommissionValue).multiply(percent).divide(new BigDecimal(100)); //取平日价
    if(weekendCommissionValue==null||weekendPrice==null){
        weekEnd = base;
    }else{
        weekEnd=new BigDecimal(weekendCommissionValue).multiply(percent).divide(new BigDecimal(100));   //取周末价
    }
}
尽量使用成熟的apache的包。 必须用StringUtils.equals(first,second)判断两个字符串是否相同 StringUtils.spit来切换字符串 比如：
if(map==null || map.size()==0){
}
必须用：
if(MapUtils.isEmpty(map)){
}
if判断逻辑过于复杂的必须单独抽取到一个可以理解的方法 比如
if (APEX_ACCOUNT_STATUS_PENDING.equals(status)
        ||APEX_ACCOUNT_STATUS_ACCOUNT_SETUP.equals(status)
        ||APEX_ACCOUNT_STATUS_ACTION_REQUIRED.equals(status)
        ||APEX_ACCOUNT_STATUS_BACK_OFFICE.equals(status)){
    updateStatus = UsAccountConstants.ACCOUNT_UPDATE_STATUS_APPROVING;
}else if (APEX_ACCOUNT_STATUS_CANCELED.equals(status)
        ||APEX_ACCOUNT_STATUS_REJECTED.equals(status)
        ||APEX_ACCOUNT_STATUS_ERROR.equals(status)){
    updateStatus = UsAccountConstants.ACCOUNT_UPDATE_STATUS_REJECT;
}else if (APEX_ACCOUNT_STATUS_COMPLETE.equals(status)){
    updateStatus = UsAccountConstants.ACCOUNT_UPDATE_STATUS_PASS;
} else {
    throw new IllegalStateException();
}
return updateStatus;
可以写成
if (underReview()){
    apexStatus = UsAccountConstants.APEX_VERIFY_STATUS_APPROVING;
}else if (isRejected()){
    apexStatus = UsAccountConstants.APEX_VERIFY_STATUS_REJECT;
}else if (isPassApexAudit()){
    apexStatus = UsAccountConstants.APEX_VERIFY_STATUS_PASS;
} else {
    throw new IllegalStateException();
}
return apexStatus;
面向接口编程，尽量少用静态方法，加强代码的可测试性。
合理运用各种设计模式。
遗留代码适时重构。
不使用通配符 Imports 示例代码：
// Bad.
// - Where did Foo come from?
import com.twitter.baz.foo.*;
import com.twitter.*;

interface Bar extends Foo {
  ...
}

// Good.
import com.twitter.baz.foo.BazFoo;
import com.twitter.Foo;

interface Bar extends Foo {
  ...
}
IntelliJ IDEA 设置： IDEA SETTING
无资源泄露和线程安全问题
线程上下文变量的设置与清除必须配对。
静态方法（比如常见的Util） 或单例必须是线程安全的。
SimpleDateFormat 类是非线程安全的，每次使用都 要重新 构造。推荐使用apache的DateFormatUtils和DateUtils。
为记录加锁时，需要保持一致的加锁顺序，否则可能会死锁
对系统资源的访问，使用后必须释放。包括：文件流 、线程、网络连接 、数据库连接等。
远程调用设置合理的超时时间，以及超时的降级处理。
外部资源的路径调用都应当采用maven 的 profile配置方式来控制，必须区分 Dev、beta、product （此项需要运维同学配合）
使用公司推荐的代码实现方式
数据库规范
事务越短越好，禁止在事务中做远程耗时调用。
日志规范
打印日志要能唯一标识出代码运行的业务，对于接口，建议打印出接口的入参。 代码范例：
if (logger.isInfoEnabled()) {
    logger.info("根据编号加载产品请求[code={}]",code);
}
对于捕获的异常信息，必须将异常堆栈打印出。 正确的代码范例：
logger.error("XXXXX查询系统异常。参数:"调用参数信息或者调用上下文链标识 , e);
错误的代码范例：
logger.error("XXXXX查询系统异常:" + e.getMessage());
日志代码不能影响业务主流程。 错误的代码范例：
result=null;
logger.warn("根据编号加载产品，产品编号为空，result.getCode={}" , result.getCode());
日志不允许打印用户的密码，账户等敏感信息
禁止使用System.out输出日志，一律必须使用日志工具类logger打印日志
监控
所有的异常和业务异常必须有监控和报警。
tomcat的线程池必须有监控（上限200系统就挂了）。
提供接口的响应时间和调用次数，出错次数。
所依赖接口的调用次数，响应时间，出错次数。
jvm监控
添加监控原则
工期没有特别紧急要求的情况下，监控、报警必须在功能上线时添加完成
对于上线比较紧急的情况，上线完成后24小时内，必须完成监控、报警功能添加
开发跟进，QA检查
安全问题
安全规范
请遵守安全部门的安全规范, 下面几点是需要重点关注的。
敏感信息保护 用户的敏感信息包括密码、短信验证码、支付验证码、身份证号、银行卡号、银行密钥，商户密钥等信息；用户敏感信息丌能泄露，否则可能会带来不安全因素。可能会导致敏感信息泄露的方式有:Logger、URL的get参数（因为 URL 的get 参数会在 nginx和tomcat 日志中被输出）。
web安全
对于前台的 web页面，必须加上表单防重复提交功能；
对于业务关键字段需要加上防篡改的功能，否则可能造成业务重复执行或者被客户端恶意修改。 （比如订单提交）
对于前台的web页面，严禁出现方便调试的后门页面。
订单的匿名查询（用手机号和订单号）页面，必须使用token机制，防止被流氓搜索引擎索引而泄漏用户信息。
关于XSS漏洞的防范
用户身份证、手机号码要进行加密。具体规则请咨询产品和安全部门。
对有权限的验证，除了功能权限(uri)的判断外，还需要特别注意数据权限，防止用户更改浏览器的id而访问或者修改不属于自己的数据。
接口要明确谁会调用，应当有ip 或 token 校验机制
公司内部的调用，同机房的应当走内网域名调用
