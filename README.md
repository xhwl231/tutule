# tutule
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;
using System.Linq;
using Newtonsoft.Json.Linq;
using UnityEngine.SceneManagement;
using UnityEngine.UI;

public class DataManager : MonoBehaviour
{
    /// <summary>
    /// 服务器地址
    /// </summary>
    public string serverPath;

    /// <summary>
    /// 轮播图表
    /// </summary>
    public Dictionary<int, SlideshowData> slideshowDict = new Dictionary<int, SlideshowData>();

    /// <summary>
    /// 首页列表
    /// </summary>
    public Dictionary<int, HomePageListData> homePageDict = new Dictionary<int, HomePageListData>();

    /// <summary>
    /// 消息
    /// </summary>
    public Dictionary<int, MessageData> messageDict = new Dictionary<int, MessageData>();

    /// <summary>
    /// 历史记录
    /// </summary>
    public Dictionary<int, HistoryData> historyDict = new Dictionary<int, HistoryData>();

    /// <summary>
    /// 所有AR
    /// </summary>
    public Dictionary<int, AllARData> allARDict = new Dictionary<int, AllARData>();

    /// <summary>
    /// 首页更多
    /// </summary>
    public Dictionary<string, HomePageMoreData> homeMoreDict = new Dictionary<string, HomePageMoreData>();

    /// <summary>
    /// 加载界面状态
    /// </summary>
    public bool loadingPanelState;

    void Start()
    {
        if (!Directory.Exists(Application.persistentDataPath+"/AssetBundles/"))
        {
            Directory.CreateDirectory(Application.persistentDataPath + "/AssetBundles/");
        }
        StartCoroutine(IELoadData(1, "GetCarousel"));
        StartCoroutine(IELoadData(2, "HomePage"));
		Screen.SetResolution (1080,1920,true);
		Screen.orientation = ScreenOrientation.Portrait;
		Screen.sleepTimeout = SleepTimeout.NeverSleep;
    }

    /// <summary>
    /// 显示图片
    /// </summary>
    /// <param name="order"></param>
    /// 选择界面图片加载选项
    private void ShowUI(int order)
    {
        switch (order)
        {
            case 1://轮播图
                StartCoroutine(IEShowSlideShow());
                break;
            case 2://首页列表
                StartCoroutine(IEShowHomePageList());
                break;
            case 3://消息中心
                UIMessagePanel messagePanel = FindObjectOfType<UIMessagePanel>();
                GameObject go = messagePanel.transform.Find("Message/Viewport/Content").gameObject;
                int index = 0;
                foreach (var key in messageDict.Keys)
                {
                    GameObject messageButtonGo = Instantiate(Resources.Load("UIPrefab/MessageButton") as GameObject);
                    messageButtonGo.transform.SetParent(go.transform, false);
                    messageButtonGo.transform.Find("BgRegion/TimeText").GetComponent<Text>().text = GetMessageData(key).data[index].MessageCreateTime;
                    messageButtonGo.transform.Find("BgRegion/MessageText").GetComponent<Text>().text = GetMessageData(key).data[index].MessageContent;
                    index++;
                }
                break;
            case 4://历史记录
                StartCoroutine(IEShowHistory());
                break;
            case 5://所有AR
                StartCoroutine(IEShowAllAR());
                break;
            case 6://首页更多
                StartCoroutine(IEShowHomeMore());
                break;
        }
    }

    /// <summary>
    /// 显示轮播图
    /// </summary>
    /// <returns></returns>
    private IEnumerator IEShowSlideShow()
    {
        Carousel ca = FindObjectOfType<Carousel>();
        Image[] img = new Image[ca.transform.childCount];
        for (int i = 0; i < img.Length; i++)
        {
            img[i] = ca.transform.GetChild(i).GetComponent<Image>();
        }
        int index = 0;
        foreach (var key in slideshowDict.Keys)
        {
            if (index < img.Length)
            {
                LoadDataControl.Instance.SetAsyncImage(GetSlideshowData(key).data[index ].CarouselPath, img[index++]);
                yield return new WaitForEndOfFrame();
            }
        }
        StopCoroutine(IEShowSlideShow());
    }

    /// <summary>
    /// 显示首页列表UI
    /// 协程解决首页界面不能滑动问题
    /// </summary>
    /// <returns></returns>
    private IEnumerator IEShowHomePageList()
    {
        UIMainPanel mainPanel = FindObjectOfType<UIMainPanel>();
        GameObject go = mainPanel.transform.Find("Advertising/Viewport/Content/Commodity").gameObject;
        int index = 0;
        foreach (var key in homePageDict.Keys)
        {
            GameObject chooseItemGo = Instantiate(Resources.Load("UIPrefab/ChooseItem") as GameObject);
			chooseItemGo.GetComponent <RectTransform > ().sizeDelta = new Vector2 (480, 210); 
            chooseItemGo.GetComponent<ChooseItem>().bookType = GetHomePageListData(key).data[index].ID;
            chooseItemGo.transform.SetParent(go.transform, false);
            chooseItemGo.transform.Find("Tittlebar/Text").GetComponent<Text>().text = GetHomePageListData(key).data[index].BookName;
            int indexPic = 0;
            for (int i = 0; i < homePageDict[key].data[index].PicList.Length; i++)
            {
                for (int j = 0; j < homePageDict[key].data[index].PicList[indexPic].VRList.Length; j++)
                {
                    yield return new WaitForEndOfFrame();
                    GameObject advertisingButtonGo = Instantiate(Resources.Load("UIPrefab/AdvertisingButton") as GameObject);
                    advertisingButtonGo.transform.SetParent(chooseItemGo.transform.Find("ChooseItemScrollRect"), false);
                    advertisingButtonGo.GetComponent<AdvertisingButton>().taobaoUrl = GetHomePageListData(key).data[index].PicList[indexPic].VRList[j].TaobaoUrl;
                    LoadDataControl.Instance.SetAsyncImage(GetHomePageListData(key).data[index].PicList[indexPic].VRList[j].PicUrl, advertisingButtonGo.GetComponent<Image>());
                }
                indexPic++;
            }
            index++;
        }

        go.GetComponent<ContentSizeFitter>().enabled = false;
        yield return new WaitForEndOfFrame();
        go.GetComponent<ContentSizeFitter>().enabled = true;
        yield return new WaitForEndOfFrame();
        go.GetComponent<ContentSizeFitter>().enabled = false;
        yield return new WaitForEndOfFrame();
        go.GetComponent<ContentSizeFitter>().enabled = true;
        StopCoroutine(IEShowHomePageList());
    }
    /// <summary>
    /// 显示所有AR的UI
    /// </summary>
    /// <returns></returns>
    private IEnumerator IEShowAllAR()
    {
        UIAllARPanel allArPanel = FindObjectOfType<UIAllARPanel>();
        GameObject go = allArPanel.transform.Find("History/Viewport/Content").gameObject;
        int index = 0;
        foreach (var key in allARDict.Keys)
        {
            yield return new WaitForEndOfFrame();
            GameObject allARItemGo = Instantiate(Resources.Load("UIPrefab/AllARItem") as GameObject);
            AllARItem allArItem = allARItemGo.AddComponent<AllARItem>();
            allArItem.id = GetAllARData(key).data[index].ID;
            allArItem.picUrl = GetAllARData(key).data[index].BookPicture;
			allArItem.bookname = GetAllARData (key).data [index].BookName ;
            allARItemGo.transform.SetParent(go.transform, false);
            allARItemGo.transform.Find("ItemNameText1").GetComponent<Text>().text = GetAllARData(key).data[index].BookName;
            allARItemGo.transform.Find("ItemNameText2").GetComponent<Text>().text = GetAllARData(key).data[index].BookName;
            LoadDataControl.Instance.SetAsyncImage(GetAllARData(key).data[index].BookPicture,allARItemGo.transform.Find("ItemImage").GetComponent<Image>());
            index++;
        }
        StopCoroutine(IEShowAllAR());
    }

    /// <summary>
    /// 显示历史记录
    /// </summary>
    /// <returns></returns>
    private IEnumerator IEShowHistory()
    {
        UIMyARPanel myARPanel = FindObjectOfType<UIMyARPanel>();
        GameObject go = myARPanel.transform.Find("History/Viewport/Content").gameObject;
        int indexHis = 0;
        foreach (var key in historyDict.Keys)
        {
            GameObject historyItemGo = Instantiate(Resources.Load("UIPrefab/HistoryItem") as GameObject);
            historyItemGo.transform.SetParent(go.transform, false);
            historyItemGo.transform.Find("Text").GetComponent<Text>().text = GetHistoryData(key).historyList[indexHis].CreateTime;
            for (int i = 0; i < GetHistoryData(key).historyList[indexHis].data.Length; i++)
            {
                yield return new WaitForEndOfFrame();
                GameObject historyButtonGo = Instantiate(Resources.Load("UIPrefab/HistoryButton") as GameObject);
                historyButtonGo.transform.SetParent(historyItemGo.transform.Find("ChooseItemScrollRect"), false);
                historyButtonGo.transform.Find("Text").GetComponent<Text>().text = GetHistoryData(key).historyList[indexHis].data[i].PicName;
                LoadDataControl.Instance.SetAsyncImage(GetHistoryData(key).historyList[indexHis].data[i].PicUrl, historyButtonGo.transform.Find("Image").GetComponent<Image>());
            }
            indexHis++;
        }
        go.GetComponent<ContentSizeFitter>().enabled = false;
        yield return new WaitForEndOfFrame();
        go.GetComponent<ContentSizeFitter>().enabled = true;
        StopCoroutine(IEShowHistory());
    }

    /// <summary>
    /// 首页更多
    /// </summary>
    /// <returns></returns>
    private IEnumerator IEShowHomeMore()
    {
        UIAllARPanel allArPanel = FindObjectOfType<UIAllARPanel>();
        allArPanel.transform.Find("Tittle/Text").GetComponent<Text>().text = recordBookName;
        GameObject go = allArPanel.transform.Find("History/Viewport/Content").gameObject;
        GameObject moreItemGo = Instantiate(Resources.Load("UIPrefab/MoreItem") as GameObject);
        moreItemGo.transform.SetParent(go.transform, false);
        int index = 0;
        foreach (var key in homeMoreDict.Keys)
        {
            yield return new WaitForEndOfFrame();
            GameObject moreButtonGo = Instantiate(Resources.Load("UIPrefab/MoreButton") as GameObject);
            moreButtonGo.GetComponent<MoreButton>().taobaoUrl = GetHomeMoreData(key).data[index].TaobaoUrl;
            moreButtonGo.transform.SetParent(moreItemGo.transform, false);
            moreButtonGo.transform.Find("Text").GetComponent<Text>().text = GetHomeMoreData(key).data[index].PicName;
            LoadDataControl.Instance.SetAsyncImage(GetHomeMoreData(key).data[index].PicUrl, moreButtonGo.transform.Find("Image").GetComponent<Image>());
            index++;
        }
        StopCoroutine(IEShowHomeMore());
    }

    /// <summary>
    /// 选择存储数据类型
    /// </summary>
    /// <param name="order"></param>
    /// 选择存储数据类型选项
    /// <param name="res"></param>
    /// 传输json数据
    private void ChooseType(int order, string res)
    {
        var value = JObject.Parse(res);
        switch (order)
        {
            case 1://轮播图
                SlideshowData slideshow = new SlideshowData();
                loadingPanelState = slideshow.state = (bool)value["state"];
                if (slideshow.state)
                {
				slideshow.data = new Data_SlideshowData[value ["data"].Count ()];
                    for (int i = 0; i < value["data"].Count(); i++)
                    {
                        slideshow.data[i] = new Data_SlideshowData();
                        slideshow.data[i].ID = int.Parse(value["data"][i]["ID"].ToString());
                        slideshow.data[i].CarouselPath = value["data"][i]["CarouselPath"].ToString();
                        //slideshow.data.UrlPath = value["data"][i]["UrlPath"].ToString();
                        slideshowDict.Add(slideshow.data[i].ID, slideshow);
                    }
                }
                break;
            case 2://首页列表
                HomePageListData homePage=new HomePageListData();
                homePage.state = (bool)value["state"];
                if (homePage.state)
                {
                    homePage.data=new Data_HomePageListData[value["data"].Count()];
                    for (int i = 0; i < value["data"].Count(); i++)
                    {
                        homePage.data[i] =new Data_HomePageListData();
                        homePage.data[i].ID = int.Parse(value["data"][i]["ID"].ToString());
                        homePage.data[i].BookName = value["data"][i]["BookName"].ToString();

                        homePage.data[i].PicList=new PicList[value["data"][i]["PicList"].Count()];
                        for (int j = 0; j < value["data"][i]["PicList"].Count(); j++)
                        {
                            homePage.data[i].PicList[j]=new PicList();
                            homePage.data[i].PicList[j].VRList = new VRList[value["data"][i]["PicList"][j]["VRList"].Count()];
                            for (int k = 0; k < value["data"][i]["PicList"][j]["VRList"].Count(); k++)
                            {
                                homePage.data[i].PicList[j].VRList[k] = new VRList();
                                homePage.data[i].PicList[j].VRList[k].ID = value["data"][i]["PicList"][j]["VRList"][k]["ID"].ToString();
                                homePage.data[i].PicList[j].VRList[k].PicUrl = value["data"][i]["PicList"][j]["VRList"][k]["PicUrl"].ToString();
                                homePage.data[i].PicList[j].VRList[k].TaobaoUrl = value["data"][i]["PicList"][j]["VRList"][k]["TaobaoUrl"].ToString();
                            }
                        }
                        homePageDict.Add(homePage.data[i].ID, homePage);
                    }
                }
                break;
            case 3://消息中心
                MessageData messageData = new MessageData();
                messageData.state = (bool)value["state"];
                if (messageData.state)
                {
                    messageData.data = new Data_MessageData[value["data"].Count()];
                    for (int i = 0; i < value["data"].Count(); i++)
                    {
                        messageData.data[i] = new Data_MessageData();
                        messageData.data[i].ID = int.Parse(value["data"][i]["ID"].ToString());
                        messageData.data[i].MessageTitle = value["data"][i]["MessageTitle"].ToString();
                        messageData.data[i].MessageContent = value["data"][i]["MessageContent"].ToString();
                        messageData.data[i].MessageCreateTime = value["data"][i]["MessageCreateTime"].ToString();
                        messageData.data[i].rowNum = int.Parse(value["data"][i]["rowNum"].ToString());
                        messageDict.Add(messageData.data[i].ID, messageData);
                    }
                }
                break;
            case 4://历史记录
                HistoryData historyData = new HistoryData();
                historyData.historyList = new HistoryList[value["HistoryList"].Count()];
                for (int i = 0; i < value["HistoryList"].Count(); i++)
                {
                    historyData.historyList[i] = new HistoryList();
                    historyData.historyList[i].ID = int.Parse(value["HistoryList"][i]["ID"].ToString());
                    historyData.historyList[i].CreateTime = value["HistoryList"][i]["CreateTime"].ToString();
                    historyData.historyList[i].data = new Data_BookType[value["HistoryList"][i]["data"].Count()];
                    for (int j = 0; j < value["HistoryList"][i]["data"].Count(); j++)
                    {
                        historyData.historyList[i].data[j] = new Data_BookType();
                        historyData.historyList[i].data[j].PicID = int.Parse(value["HistoryList"][i]["data"][j]["PicID"].ToString());
                        historyData.historyList[i].data[j].PicUrl = value["HistoryList"][i]["data"][j]["PicUrl"].ToString();
                        historyData.historyList[i].data[j].PicName = value["HistoryList"][i]["data"][j]["PicName"].ToString();
                    }
                    historyDict.Add(historyData.historyList[i].ID, historyData);
                }
                break;
            case 5://所有AR
                AllARData allARData = new AllARData();
                allARData.state = (bool)value["state"];
                if (allARData.state)
                {
                    allARData.data = new Data_AllARData[value["data"].Count()];
                    for (int i = 0; i < value["data"].Count(); i++)
                    {
                        allARData.data[i] = new Data_AllARData();
                        allARData.data[i].ID = int.Parse(value["data"][i]["ID"].ToString());
                        allARData.data[i].BookName = value["data"][i]["BookName"].ToString();
                        allARData.data[i].CreateTime = value["data"][i]["CreateTime"].ToString();
                        allARData.data[i].BookPicture = value["data"][i]["BookPicture"].ToString();
                        allARData.data[i].ModelUrl = value["data"][i]["ModelUrl"].ToString();
                        allARData.data[i].rowNum = int.Parse(value["data"][i]["rowNum"].ToString());
                        allARDict.Add(allARData.data[i].ID, allARData);
                    }
                }
                break;
            case 6://首页更多
                HomePageMoreData homePageMoreData = new HomePageMoreData();
                homePageMoreData.state = (bool)value["state"];
                if (homePageMoreData.state)
                {
                    homePageMoreData.data = new Data_HomePageMoreData[value["data"].Count()];
                    for (int i = 0; i < value["data"].Count(); i++)
                    {
                        homePageMoreData.data[i] = new Data_HomePageMoreData();
                        homePageMoreData.data[i].ID = value["data"][i]["ID"].ToString();
                        homePageMoreData.data[i].PicUrl = value["data"][i]["PicUrl"].ToString();
                        homePageMoreData.data[i].PicName = value["data"][i]["PicName"].ToString();
                        homePageMoreData.data[i].CreateTime = value["data"][i]["CreateTime"].ToString();
                        homePageMoreData.data[i].TaobaoUrl = value["data"][i]["TaobaoUrl"].ToString();
                        homePageMoreData.data[i].rowNum = int.Parse(value["data"][i]["rowNum"].ToString());
                        homeMoreDict.Add(homePageMoreData.data[i].ID, homePageMoreData);
                    }
                }
                break;
        }
    }

    #region 加载数据
    /// <summary>
    /// 加载轮播图，首页列表
    /// </summary>
    /// <param name="order"></param>
    /// 选项
    /// <param name="method"></param>
    /// 密码
    /// <returns></returns>
    IEnumerator IELoadData(int order, string method)
    {
        WWWForm wf = new WWWForm();
        wf.AddField("method", method);
        WWW www = new WWW(serverPath + "/Areas/SchoolAgent/IList.ashx", wf);
        yield return www;
        ChooseType(order, www.text);
        ShowUI(order);
        www.Dispose();
        StopCoroutine(IELoadData(order, method));
        Resources.UnloadUnusedAssets();
    }

    /// <summary>
    /// 登录注册
    /// </summary>
    /// <param name="method"></param>
    /// 密码
    /// <param name="data"></param>
    /// 数据
    /// <param name="islock"></param>
    /// 是否注册
    /// <returns></returns>
    public IEnumerator IELoadData(string method, string data, bool islock)
    {
        WWWForm wf = new WWWForm();
        wf.AddField("method", method);
        wf.AddField("data", data);
        WWW www = new WWW(serverPath + "/Areas/SchoolAgent/IList.ashx", wf);
        yield return www;
        logn log = FindObjectOfType<logn>();
        string[] str = www.text.Split('"');
        if (str[3] == "'用户名已存在'")
        {
            log.error.text = "用户名已存在";
        }
        else if (str[3] == "fail")
        {
            log.error.text = "用户名或密码错误";
        }
        else if (islock == false)
        {
            log.a = false;
            log.games[0].SetActive(false);
            log.game[0].SetActive(false);
            log.game[1].SetActive(true);
            log.username.text = str[5];
            log.error.text = " ";
            PlayerPrefs.SetString("username", log.inputfield2[0].text);
            PlayerPrefs.SetString("passwold", log.inputfield2[1].text);
        }
        else
        {
            log.username.text = log.inputfield[0].text;
            log.games[1].SetActive(false);
            log.game[0].SetActive(false);
            log.game[1].SetActive(true);
            log.error.text = " ";
            PlayerPrefs.SetString("username", log.inputfield[0].text);
            PlayerPrefs.SetString("passwold", log.inputfield[1].text);
        }
        www.Dispose();
        StopCoroutine(IELoadData(method, data, islock));
        Resources.UnloadUnusedAssets();
    }

    /// <summary>
    /// 加载所有AR
    /// </summary>
    /// <param name="order"></param>
    /// 选项
    /// <param name="method"></param>
    /// 密码
    /// <param name="pageIndex"></param>
    /// 密码-页数
    /// <param name="pageSize"></param>
    /// 密码-内容
    /// <returns></returns>
    public IEnumerator IELoadData(int order, string method, int pageIndex,int pageSize)
    {
        WWWForm wf = new WWWForm();
        wf.AddField("method", method);
        wf.AddField("pageIndex", pageIndex);
        wf.AddField("pageSize", pageSize);
        WWW www = new WWW(serverPath + "/Areas/SchoolAgent/IList.ashx", wf);
        yield return www;
        ChooseType(order, www.text);
        ShowUI(order);
        www.Dispose();
        StopCoroutine(IELoadData(order, method, pageIndex, pageSize));
        Resources.UnloadUnusedAssets();
    }

    /// <summary>
    /// 加载首页更多
    /// </summary>
    /// <param name="order"></param>
    /// 选项
    /// <param name="method"></param>
    /// 密码
    /// <param name="bookType"></param>
    /// 密码-书籍类型
    /// <param name="pageIndex"></param>
    /// 密码-页数
    /// <param name="pageSize"></param>
    /// 密码-内容
    /// <returns></returns>
    private string recordBookName;
    public IEnumerator IELoadData(int order, string method, int bookType, int pageIndex, int pageSize ,string bookName = null)
    {
        WWWForm wf = new WWWForm();
        wf.AddField("method", method);
        wf.AddField("BookType", bookType);
        wf.AddField("pageIndex", pageIndex);
        wf.AddField("pageSize", pageSize);
        WWW www = new WWW(serverPath + "/Areas/SchoolAgent/IList.ashx", wf);
        yield return www;
        recordBookName = bookName;
		Debug.Log (bookName );
        ChooseType(order, www.text);
        ShowUI(order);
        www.Dispose();
        StopCoroutine(IELoadData(order, method, bookType, pageIndex, pageSize, bookName));
        Resources.UnloadUnusedAssets();
    }

    /// <summary>
    /// 加载历史记录
    /// </summary>
    /// <param name="order"></param>
    /// 选项
    /// <param name="isLock"></param>
    /// 锁住显示UI
    public void LoadHistoryToJson(int order, bool isLock = false)
    {
        string path = Application.persistentDataPath + "/JsonData/HistoryData.json";
        if (File.Exists(path))
        {
            StreamReader streamReader=new StreamReader(path);
            string data = streamReader.ReadToEnd();
            ChooseType(order, data);
            if (!isLock)
            {
                ShowUI(order);
            }
            streamReader.Close();
            streamReader.Dispose();
        }
    }

    #endregion


    #region 得到数据
    /// <summary>
    /// 得到轮播图数据
    /// </summary>
    /// <returns></returns>
    public SlideshowData GetSlideshowData(int id)
    {
        SlideshowData slideshow=new SlideshowData();
        if (slideshowDict.TryGetValue(id, out slideshow))
        {
            return slideshow;
        }
        Debug.LogError("轮播图数据不存在");
        return null;
    }
    /// <summary>
    /// 得到首页列表图数据
    /// </summary>
    /// <param name="id"></param>
    /// key值
    /// <returns></returns>
    public HomePageListData GetHomePageListData(int id)
    {
        HomePageListData homePage = new HomePageListData();
        if (homePageDict.TryGetValue(id, out homePage))
        {
            return homePage;
        }
        Debug.LogError("首页列表数据不存在");
        return null;
    }
    /// <summary>
    /// 消息数据
    /// </summary>
    /// <param name="id"></param>
    /// key值
    /// <returns></returns>
    public MessageData GetMessageData(int id)
    {
        MessageData messageData = new MessageData();
        if (messageDict.TryGetValue(id, out messageData))
        {
            return messageData;
        }
        Debug.LogError("消息数据不存在");
        return null;
    }
    /// <summary>
    /// 历史记录
    /// </summary>
    /// <param name="id"></param>
    /// key值
    /// <returns></returns>
    public HistoryData GetHistoryData(int id)
    {
        HistoryData historyData = new HistoryData();
        if (historyDict.TryGetValue(id, out historyData))
        {
            return historyData;
        }
        Debug.LogError("历史记录数据不存在");
        return null;
    }
    /// <summary>
    /// 所有AR
    /// </summary>
    /// <param name="id"></param>
    /// key值
    /// <returns></returns>
    public AllARData GetAllARData(int id)
    {
        AllARData allARData = new AllARData();
        if (allARDict.TryGetValue(id, out allARData))
        {
            return allARData;
        }
        Debug.LogError("所有AR数据不存在");
        return null;
    }
    /// <summary>
    /// 首页更多
    /// </summary>
    /// <param name="id"></param>
    /// key值
    /// <returns></returns>
    public HomePageMoreData GetHomeMoreData(string id)
    {
        HomePageMoreData homePageMoreData = new HomePageMoreData();
        if (homeMoreDict.TryGetValue(id, out homePageMoreData))
        {
            return homePageMoreData;
        }
        Debug.LogError("首页更多数据不存在");
        return null;
    }
    #endregion
}
