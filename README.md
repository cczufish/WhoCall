WhoCall - 谁CALL我 ![](_images/AppIcon-60.png?raw=true)
=======

iOS来电信息语音提醒，无需越狱。（需要iOS 7.0及以上版本。)

骚扰电话预警、来电归属地提醒、联系人姓名播报，这些~~有中国特色~~人性化的电话功能，iOS上也应该有！

![](_images/screenshot-1.png?raw=true)

功能介绍
-------

那个陌生的来电号码是我的快递来了？是卖保险的？还是骗钱的电话？一听就知道！“谁CALL我”自动查询来电号码详细信息，在响铃的同时通过语音念给你听，让你接电话前心中有数。尤其适用于戴耳机的时候，不用掏出手机就能知道是谁打来电话。

超级简单易用，点两下开关即可完成设置。然后就可以把我忘掉，我会默默保护你。

* 骚扰电话预警 - 广告推销电话、诈骗电话、骚扰电话预警，还有部分快递号码、中介号码也会提醒。
* 来电归属地提醒 - 收录最新的全国手机号码归属地+各省市固话区号数据。
* 联系人姓名播报 - 如果是号码簿中的联系人来电，会在响铃的同时念出联系人姓名，防止漏接重要电话。

注：“骚扰电话预警”和“来电归属地”功能仅对中国大陆地区电话号码有效。


给开发者看的
-------

不要试图把这个App提交到App Store，我试过，不行，所以才干脆开源了。

此App使用了私有API获取来电号码，虽然API的调用经过伪装，能绕过自动检测，但是审核员会对此类App做特别关照，仍然有办法查出来调用的私有API。另外App常驻后台的做法也可能违反审核条例。

以下代码可能对你有用：

* `WCCallCenter` - 展示了如何用`dlsym`调用私有的C接口，并对函数名字符串做简单的加密，以绕过App提交过程中的自动检查。
* `WCLiarPhoneList` - 通过百度搜索电话号码，判断电话是否是骚扰电话，并提取出具体的类型（广告推销、诈骗……）。
* `WCPhoneLocator` - 电话号码归属地查询。


License
-------
You may use this project under the terms of the MIT License.


Acknowledgement
--------
* [FMDB](https://github.com/ccgus/fmdb)
* [UIKitCategoryAdditions](https://github.com/MugunthKumar/UIKitCategoryAdditions)
* [MMPDeepSleepPreventer](https://github.com/mruegenberg/MMPDeepSleepPreventer)
* [moquery](https://github.com/roymax/moquery)
* 





做了点修改

    // 检查归属地
    void (^checkPhoneLocation)(void) = ^{
        if (self.handlePhoneLocation && !isContact) {
            NSString *location = [[WCPhoneLocator sharedLocator] locationForPhoneNumber:number];
            if (location) {
                NSString *addr = [self isMobileNumber:number];
                // 注意格式，除了地址，还可以有“本地”等
                NSString *msg = [NSString stringWithFormat:@"%@%@电话", location,addr];
                
                [self notifyMessage:msg forPhoneNumber:number];
            }
        }
    };

// 正则判断手机号码
- (NSString *)isMobileNumber:(NSString *)mobileNum
{
    /**
     * 手机号码
     * 移动：134[0-8],135,136,137,138,139,150,151,157,158,159,182,187,188
     * 联通：130,131,132,152,155,156,185,186
     * 电信：133,1349,153,180,189
     */
    NSString * MOBILE = @"^1(3[0-9]|5[0-35-9]|8[025-9])\\d{8}$";
    /**
     10         * 中国移动：China Mobile
     11         * 134[0-8],135,136,137,138,139,150,151,157,158,159,182,187,188
     12         */
    NSString * CM = @"^1(34[0-8]|(3[5-9]|5[017-9]|8[278])\\d)\\d{7}$";
    /**
     15         * 中国联通：China Unicom
     16         * 130,131,132,152,155,156,185,186
     17         */
    NSString * CU = @"^1(3[0-2]|5[256]|8[56])\\d{8}$";
    /**
     20         * 中国电信：China Telecom
     21         * 133,1349,153,180,189
     22         */
    NSString * CT = @"^1((33|53|8[09])[0-9]|349)\\d{7}$";
    /**
     25         * 大陆地区固话及小灵通
     26         * 区号：010,020,021,022,023,024,025,027,028,029
     27         * 号码：七位或八位
     28         */
    // NSString * PHS = @"^0(10|2[0-5789]|\\d{3})\\d{7,8}$";
    
    NSPredicate *regextestmobile = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", MOBILE];
    NSPredicate *regextestcm = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CM];
    NSPredicate *regextestcu = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CU];
    NSPredicate *regextestct = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CT];
    
    if (([regextestmobile evaluateWithObject:mobileNum] == YES)&& ([regextestcm evaluateWithObject:mobileNum] == YES))
    {
        return @"移动";
    }
    else if (([regextestmobile evaluateWithObject:mobileNum] == YES)&& ([regextestcu evaluateWithObject:mobileNum] == YES))
    {
        return @"联通";
    }else if (([regextestmobile evaluateWithObject:mobileNum] == YES)&& ([regextestct evaluateWithObject:mobileNum] == YES))
    {
        return @"电信";
    }else{
        return @"固话或者小灵通";
    }
}
