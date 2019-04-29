---
title:       "SpringMVC封装常用的CRUD操作"
subtitle:    ""
description: ""
date:        2019-04-09
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-06-04-introducing-the-istio-v1alpha3-routing-api/background.jpg"
tags:        ["java基础", "基础框架", "常规CRUD"]
categories:  ["Tech" ]
---

[TOC]

目标：  主要处理单表的进行列表展示， 和 增加，删除，修改，查询等这样的操作的封装处理。对于我们用的myBiatis来处理数据库， 都是通过插件， 数据库和实体以及DAO都是自动生成的，这样的话， 我们可以发现， DAO层都是相同的方法。 同时再编写service层和controller的时候，很多的功能点都是重复的。  所以进行了封装。

主要思路： 通过泛型来控制DAO层的对应的实体类。  然后对于Service和Controller层的时候， 写成抽象类，因为通过@Autowired注册的时候， 是根据顺序来处理的，如果， 如果直接再基类中注册的话， sprng不知道你要注册的是哪一个DAO。 所以这两个类需要进行抽象话。  然后再子类中进行具体的注册处理。 当然封装的一些方法， 需要进行变动的时候，通过重写就好。   具体代码如下：

# Dao

```java
public interface SimplenessDao<K> {
	
	  int deleteByPrimaryKey(BigDecimal medId);
 
    int insert(K k);
 
    int insertSelective(K k);
 
    K selectByPrimaryKey(BigDecimal medId);
 
    int updateByPrimaryKeySelective(K k);
 
    int updateByPrimaryKey(K k);
    
    // 列表数据
    List<K> findList(Map<String,Object> map);
    
    // 列表记录 
    Integer findListCount(Map<String,Object> map);
}
```



# Service

我这边加入了ISimplenessService， 主要是用来规范和习惯， 注入到controller的我直接用了 SimplenessServiceImpl

```java
public interface ISimplenessService<K> {
	
	// 查询列表
	List<K> findList(Map<String,Object> paramsMap);
	
	// 查询记录
	Integer findListCount(Map<String,Object> paramsMap);
	
	// 添加和修改--这个用于ajax处理
	HttpAjaxResult  saveOrUpdate(K k,Integer id);
	
	// 查询id对应对象 
	K  findEntityByParams(Map<String,Integer> paramsMap,Class<K> clazz);
	
	// 根据id删除对象 
	HttpAjaxResult delByPrimKey(int id);
}
```



```java

@SuppressWarnings("rawtypes")
public abstract class SimplenessServiceImpl<K> implements ISimplenessService<K> {
 
      public abstract SimplenessDao DaoHandler();

      @SuppressWarnings("unchecked")
      public K selectByPrimaryKey(BigDecimal medId) {

        return (K) DaoHandler().selectByPrimaryKey(medId);
      }
 
      @SuppressWarnings("unchecked")
      @Override
      public List findList(Map paramsMap) {

        return DaoHandler().findList(paramsMap);
      }

      @SuppressWarnings("unchecked")
      @Override
      public Integer findListCount(Map paramsMap) {

        return DaoHandler().findListCount(paramsMap);
      }
 
      @SuppressWarnings("unchecked")
      @Override
      public HttpAjaxResult saveOrUpdate(K k, Integer id) {
        int count = 0;
        k = this.dataSupplementHandler(k,id);
        if (id == null || id == 0) {
          count = DaoHandler().insertSelective(k);
        } else {
          count = DaoHandler().updateByPrimaryKeySelective(k);
        }
        return this.ajaxBusiness(count);
      }

      /***
       * 
      * @Title: dataSupplement 
      * @Description: TODO(这里用一句话描述这个方法的作用)  数据补充处理 
      * @param @param k    设定文件 
      * @return void    返回类型 
      * @throws
       */
      public K  dataSupplementHandler(K k,Integer id){

        return k;
      }
	
 
      @Override
      public HttpAjaxResult delByPrimKey(int id) {
        BigDecimal ids = new BigDecimal(id);
        int count = DaoHandler().deleteByPrimaryKey(ids);
        return this.ajaxBusiness(count);
      }

      /**
       * 
       * @Title: ajaxBusiness
       * @Description: TODO(这里用一句话描述这个方法的作用) ajax统一回复处理
       * @param @param count
       * @param @return 设定文件
       * @return HttpAjaxResult 返回类型
       * @throws
       */
      public HttpAjaxResult ajaxBusiness(int count) {
        HttpAjaxResult result = new HttpAjaxResult();
        if (count > 0) {
          result.setResultCode(200);
          result.setResultMsg("success");
        } else {
          result.setResultCode(400);
          result.setResultMsg("fail");
        }
        return result;
      }

      @SuppressWarnings("unchecked")
      public K findEntityByParams(Map paramsMap, Class clazz) {
        try {
          K k = null;
          if (paramsMap != null) {
            boolean flag = paramsMap.containsKey("id");
            if (flag) {
              int id = (int) paramsMap.get("id");
              if (id == 0) {
                // 创建对象。
                k = (K) clazz.newInstance();
              } else {
                BigDecimal decId = new BigDecimal(id);
                k = (K) DaoHandler().selectByPrimaryKey(decId);
              }
              return k;
            }
          }
          return k;
        } catch (Exception e) {
          e.printStackTrace();
        }
        return null;
      }
}
```



# Controller

```java
@SuppressWarnings("rawtypes")
public abstract class SimplenessController<K> extends BaseController{
	  
      // 外面填充
      public abstract SimplenessServiceImpl  serviceImplHandler();
      /***
       * 
      * @Title: setUpUrlPrefix 
      * @Description: TODO(这里用一句话描述这个方法的作用)  统一访问请求URL
      * @param @return    设定文件 
      * @return String    返回类型 
      * @throws
       */
      public abstract String  setUpUrlPrefix();

	
      /***
       * 
      * @Title: listView 
      * @Description: TODO(这里用一句话描述这个方法的作用)  列表展示页面 
      * @param @return    设定文件 
      * @return ModelAndView    返回类型 
      * @throws
       */
      @RequestMapping(value = "/listView")
      public  ModelAndView listView(){
        ModelAndView  modelView = this.pageForward(this.setUpUrlPrefix()+"/list"); 
        return  modelView;
      };


      /***
       * 
      * @Title: listData 
      * @Description: TODO(这里用一句话描述这个方法的作用)  ajax  list数据返回 
      * @param @param pageBean
      * @param @throws Exception    设定文件 
      * @return void    返回类型 
      * @throws
       */
      @RequestMapping(value = "/listData")
      @ResponseBody
      public void listData(PageBean pageBean,K k) throws Exception{
        Map<String, Object> map = new HashMap<String, Object>();
        map = this.addParamsBusiness(k,map);
        List dataList = serviceImplHandler().findList(map);
          int  pageCount = serviceImplHandler().findListCount(map);
          listDataResorce(map,pageBean,dataList,pageCount);
      };


      @SuppressWarnings("unchecked")
      public void  listDataResorce(Map<String, Object> map,PageBean pageBean,List dataList,int pageCount) throws Exception{
        map.put("pageBean", pageBean);
          ResPageBean resPageBean = new ResPageBean(pageBean.getsEcho(), pageCount,pageCount);
        resPageBean.setAaData(dataList);
        this.outputStr("yyyy-MM-dd HH:mm:ss", resPageBean);
      }


      /**
       * 
      * @Title: addParamsBusiness 
      * @Description: TODO(这里用一句话描述这个方法的作用)  添加查询条件处理。  
      * @param @param map
      * @param @return    设定文件 
      * @return Map<String,Object>    返回类型 
      * @throws
       */
        public Map<String, Object> addParamsBusiness(K k,Map<String, Object> map){


        return map;
      }


         // 设置这个Class类型， 用来实现增加操作问题。 (添加的时候需要创建对象用)
         public abstract Class<K> setUpClass();

        /***
         * 
        * @Title: addView 
        * @Description: TODO(这里用一句话描述这个方法的作用)      编辑和新增页面返回
        * @param @param id
        * @param @return    设定文件 
        * @return ModelAndView    返回类型 
        * @throws
         */
        @RequestMapping(value="/addOrUpdate/{id}")
      public ModelAndView addOrUpdate(@PathVariable("id") Integer id){
          Map<String, Object> map = new HashMap<String, Object>();
          map.put("id", id);
          K k = (K) serviceImplHandler().findEntityByParams(map,this.setUpClass());
        ModelAndView  modelView = this.pageForward(this.setUpUrlPrefix()+"/addOrUpdate","baseObject",k); 
        return  modelView;
        }
      /**
       * 
      * @Title: saveOrUpdate 
      * @Description: TODO(这里用一句话描述这个方法的作用)   ajax 新增和更新 
      * @param @param k
      * @param @return    设定文件 
      * @return HttpAjaxResult    返回类型 
      * @throws
       */
      @RequestMapping(value="/saveOrUpdateAjax/{id}")
      @ResponseBody
      public HttpAjaxResult saveOrUpdateAjax(K k,@PathVariable("id") Integer id){
          HttpAjaxResult ajaxResult = serviceImplHandler().saveOrUpdate(k,id);
        return  ajaxResult;
        }

        /***
         * 
        * @Title: del 
        * @Description: TODO(这里用一句话描述这个方法的作用)   ajax 删除处理 
        * @param @param id
        * @param @return    设定文件 
        * @return HttpAjaxResult    返回类型 
        * @throws
         */

        @RequestMapping(value="/del/{id}")
        @ResponseBody
      public  HttpAjaxResult del(@PathVariable("id") Integer id){
          HttpAjaxResult ajaxResult = serviceImplHandler().delByPrimKey(id);
        return  ajaxResult;
        };
}
```



# Tools

HttpAjaxResult这个类主要用来控制AJax的返回类的处理

```java
public class HttpAjaxResult {
      private int resultCode;
      private String resultMsg;
      public int getResultCode() {
        return resultCode;
      }
      public void setResultCode(int resultCode) {
        this.resultCode = resultCode;
      }
      public String getResultMsg() {
        return resultMsg;
      }
      public void setResultMsg(String resultMsg) {
        this.resultMsg = resultMsg;
      }
}
```



同时，我们再定义处理泛型的类的时候， 有时候如果为了控制对象的范围处理， 我们可以直接通过  T  extends  BaseObject  这样来进行更好的代码规范处理。 



基础代码就如上进行处理。 