---
title: O2O地图应用之判断用户订单地址是否在服务范围内
date: 2016-05-18 19:51:31
tags:
    - ASP.NET
categories: 
    - 开发笔记
description: O2O地图应用之判断用户订单地址是否在服务范围内
---

#### 需求分析

在o2o项目中，经常要用到在用户下单时判断用户所填地址的坐标点是否在服务范围内的情况，这里参考网上的实现方式，用C#来实现，经测试后有效，特此记录。

#### 代码

```
 public class MapHelper
    {

        /// <summary>
        /// 判断一个坐标点在多边形坐标点的内部还是外部
        /// </summary>
        /// <param name="point">要判断的坐标点</param>
        /// <param name="pts">多边形坐标点集合</param>
        /// <returns></returns>
        public static bool IsPointInPolygon(Point point, List<Point> pts)
        {
            int N = pts.Count;
            //如果点位于多边形的顶点或边上，也算做点在多边形内，直接返回true
            bool boundOrVertex = true;
            //经过点的次数
            int intersectCount = 0;
            double precision = 2e-10;
            Point p1, p2;
            Point p = point;//当前点

            p1 = pts[0];

            for (int i = 1; i <= N; i++)
            {
                //如果点在多边形上
                if (p.Equals(p1))
                {
                    return boundOrVertex;
                }

                p2 = pts[(i % N)];
                if (p.Lng<Math.Min(p1.Lng,p2.Lng)||p.Lng>Math.Max(p1.Lng,p2.Lng))
                {
                    p1 = p2;
                    continue;
                }

                if (p.Lng>Math.Min(p1.Lng,p2.Lng)&&p.Lng<Math.Max(p1.Lng,p2.Lng))
                {
                    if (p.Lat<=Math.Max(p1.Lat,p2.Lat))
                    {
                        if (p1.Lng==p2.Lng&&p.Lat>=Math.Min(p1.Lat,p2.Lat))
                        {
                            return boundOrVertex;
                        }

                        if (p1.Lat==p2.Lat)
                        {
                            if (p1.Lat==p.Lat)
                            {
                                return boundOrVertex;
                            }
                            else
                            {
                                intersectCount++;
                            }
                        }
                        else
                        {
                            double xinters = (p.Lng - p1.Lng) * (p2.Lat - p1.Lat) / (p2.Lng - p1.Lng) + p1.Lat;
                            if (Math.Abs(p.Lat-xinters)<precision)
                            {
                                return boundOrVertex;
                            }

                            if (p.Lat<xinters)
                            {
                                intersectCount++;
                            }
                        }
                    }
                }
                else
                {
                    if (p.Lng==p2.Lng&&p.Lat<=p2.Lat)
                    {
                        Point p3 = pts[(i+1)%N];
                        if (p.Lng>=Math.Min(p1.Lng,p3.Lng)&&p.Lng<=Math.Max(p1.Lng,p3.Lng))
                        {
                            intersectCount++;
                        }
                        else
                        {
                            intersectCount += 2;
                        }
                    }
                }
                p1 = p2;
            }

            if (intersectCount%2==0)
            {
                //偶数在多边形外
                return false;
            }
            else
            {
                //奇数在多边形内
                return true;
            }
        }
    }

    public class Point
    {
        /// <summary>
        /// 经度
        /// </summary>
        public double Lng { get; set; }
        /// <summary>
        /// 纬度
        /// </summary>
        public double Lat { get; set; }
    }
```

#### 测试 

这里我用高德地图标出了北京五环范围的坐标点集合，然后随意选择一个坐标点来进行判断：

坐标点可以用这个工具来获取：[高德地图API](http://lbs.amap.com/console/show/picker) 

**五环范围：**  

* 香泉桥   116.222208,39.992436
* 箭亭桥   116.327147,40.02046
* 上清桥   116.353948,40.02299
* 顾家庄桥 116.44128,40.020526
* 东北五环 116.48441,40.013624
* 平房桥   116.541101,39.942393
* 东南五环 116.549202,39.851595
* 旧宫新桥 116.43082,39.785968
* 狼垈东桥 116.296044,39.777442
* 宛平桥   116.225062,39.845517
* 衙门口桥 116.211308,39.894396
* 西五环   116.212595,39.944705

**随机坐标：**

* 林萃桥地铁站  116.37297,40.021857
* 望京西园四区  116.47086,39.99648
* 观音禅寺      116.533811,39.880533
* 俏狐国际      116.299713,39.772619
* 芳园里小区    116.416336,39.78394
* 润枫锦尚小区  116.429039,39.790535


```
    class Program
    {
        static void Main(string[] args)
        {
            var Plist = new List<Point> {
                new Point {Lng=116.222208,Lat= 39.992436},
                new Point {Lng=116.327147,Lat= 40.02046},
                new Point {Lng=116.353948,Lat= 40.02299},
                new Point {Lng=116.44128,Lat= 40.020526},
                new Point {Lng=116.48441,Lat=40.013624 },
                new Point {Lng=116.541101,Lat= 39.942393},
                new Point {Lng=116.549202,Lat= 39.851595},
                new Point {Lng=116.43082,Lat=39.785968},
                new Point {Lng=116.296044,Lat=39.777442 },
                new Point {Lng=116.225062,Lat=39.845517 },
                new Point {Lng=116.211308,Lat= 39.894396},
                new Point {Lng=116.212595,Lat=39.944705}
            };

            //var p = new Point { Lng = 116.37297, Lat = 40.021857 };
            //林萃桥地铁站   内

            //var p = new Point { Lng = 116.47086, Lat = 39.99648 };
            //望京西园四区  内

            //var p = new Point { Lng = 116.533811, Lat = 39.880533 };
            //观音禅寺 内

            //var p = new Point { Lng = 116.299713, Lat = 39.772619 }; 
            //俏狐国际  外

            //var p = new Point { Lng = 116.416336, Lat = 39.78394 };  
            //芳园里小区  外

            var p = new Point { Lng = 116.429039, Lat = 39.790535 };
            //润枫锦尚小区  内

            bool isin = MapHelper.IsPointInPolygon(p, Plist);
            if (isin)
            {
                Console.WriteLine("随机点在五环范围内，可以派单");
            }
            else
            {
                Console.WriteLine("随机点不在五环范围内");
            }
            Console.ReadKey();
        }
    }
```

#### 总结

* 北京的五环范围毕竟不是一个规则的多边形，可以尽量选择有标志性的坐标点来规范多边形
* 参考自：[百度地图——判断用户是否在配送范围内解决方案 - aheizi - 博客园](http://www.cnblogs.com/aheizi/p/5162992.html)
