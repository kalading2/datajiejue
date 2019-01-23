# datajiejue
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System.IO;
using System.Text.RegularExpressions;
using UnityEngine.UI;
using Newtonsoft.Json;
public class DataManager : MonoBehaviour
{
    //服务器地址
    private string serverPath;
    //保存的文件路径
    string filePath;
    //资源数据
    private string res;
    //写Json
    private StreamWriter streamWriter;
    //读Json
    private StreamReader streamReader;
    //请求
    WWW www;
    //下载的图片
    Texture2D texture2D;
    //轮播图父节点
    public Transform houseTypeImage;
    //轮播图父节点
    public Transform[] slideShowImage=new Transform[3];
    //轮播图父节点
    public Transform[] EightButtonImage=new Transform[8];
    //轮播图父节点
    public Transform RailImage;
    //单历
    public static DataManager instance;

    public UIManager UiManager;
    //存取服务器资源图片
    public Dictionary<int, FloorPlanData> FloorPlanDataDictionary = new Dictionary<int, FloorPlanData>();//房型图
    public Dictionary<int, SlideShowData> SlideShowDataDictionary = new Dictionary<int, SlideShowData>();//轮播图
    public Dictionary<int, EightButtonData> EightButtonDataDictionary = new Dictionary<int, EightButtonData>();//八个按钮
    public Dictionary<int, RailData> RailDataDictionary = new Dictionary<int, RailData>();//横条


    void Awake()
    {
        UiManager = GameObject.Find("UIRoot").GetComponent<UIManager>();
        instance = this;
        if (!Directory.Exists(Application.persistentDataPath + "/ImageCache/"))
        {
            Directory.CreateDirectory(Application.persistentDataPath + "/ImageCache/");
        }
        if (!Directory.Exists(Application.persistentDataPath + "/JsonData/"))
        {
            Directory.CreateDirectory(Application.persistentDataPath + "/JsonData/");
        }
    }

    void Start()
    {
		serverPath = "http://192.168.2.11:3504/";
        filePath = Application.persistentDataPath;
        //加载资源存入字典
        //LoadFloorPlanDataByJson();
        LoadSlideShowDataByJson();
        LoadEightButtonDataByJson();
        LoadRailDataByJson();
    }

    //获取图片地址
    private string url(int order, int rowNum)
    {
        switch (order)
        {
            case 1:
                return GetFloorPlanData(rowNum).ImgUrl;
            case 2:
                return GetSlideShowData(rowNum).ImgUrl;
            case 3:
                return GetEightButtonData(rowNum).ImgUrl;
            case 4:
                return GetRailData(rowNum).ImgUrl;
        }
        return null;
    }

    public void SetAsyncImage(string url, Image image)
    {
        //判断是否是第一次加载这张图片  
        if (!File.Exists(path + url.GetHashCode()))
        {
            //如果之前不存在缓存文件
            StartCoroutine(DownloadImage(url, image));
        }
        else
        {
            StartCoroutine(LoadLocalImage(url, path, image));
        }
    }

    IEnumerator DownloadImage(string url, Image image)
    {
        UiManager.UIProgressBarPanel.SetActive(true);
        //Debug.Log("downloading new image:" + path + url.GetHashCode());//url转换HD5作为名字  
        WWW www = new WWW(url);
        yield return www;

        if (www != null)
        {
            Texture2D tex2d = www.texture;
            //将图片保存至缓存路径
            byte[] pngData = tex2d.EncodeToPNG();
            File.WriteAllBytes(path + url.GetHashCode(), pngData);

            Sprite m_sprite = Sprite.Create(tex2d, new Rect(0, 0, tex2d.width, tex2d.height), new Vector2(0, 0));
            image.sprite = m_sprite;
            image.sprite.name = url.GetHashCode().ToString();
            LoadDataControl.Instance.asd = true;
        }
    }

    public IEnumerator LoadLocalImage(string url, string localPath, Image image)
    {
        UiManager.UIProgressBarPanel.SetActive(true);
        string filePath = "file:///" + localPath + url.GetHashCode();
        //Debug.Log("getting local image:" + filePath);
        WWW www = new WWW(filePath);
        yield return www;

        Texture2D texture = www.texture;
        Sprite m_sprite = Sprite.Create(texture, new Rect(0, 0, texture.width, texture.height), new Vector2(0, 0));
        image.sprite = m_sprite;
        image.sprite.name = url.GetHashCode().ToString();
        LoadDataControl.Instance.asd = true;
    }

    public string path
    {
        get
        {
            return Application.persistentDataPath + "/ImageCache/";
        }
    }

    //加载图片
    private void ShowDataImage(int order)
    {
        switch (order)
        {
            case 1:

                break;
            case 2:
                for (int i = 0; i < slideShowImage.Length; i++)
                {
                    SetAsyncImage(serverPath + url(2, i + 1), slideShowImage[i].GetComponent<Image>());
                }
                break;
            case 3:
                for (int i = 0; i < EightButtonImage.Length; i++)
                {
                    string str = Regex.Escape(url(3, i + 1));
                    str = str.Replace(@"\\", @"/");
                    str=str.Replace(@"\", @"");
                    SetAsyncImage(serverPath + str, EightButtonImage[i].GetComponent<Image>());
                }
                break;
            case 4:
                SetAsyncImage(serverPath + url(4, 1), RailImage.GetComponent<Image>());
                break;
        }
    }

    //写入数据
    public void WriteDataByJson(int type, string path)
    {
        if (!File.Exists(filePath + "/JsonData/" + path))
        {
            streamWriter = File.CreateText(filePath + "/JsonData/" + path);
        }
        else
        {
            streamWriter = new StreamWriter(filePath + "/JsonData/" + path);
        }
        streamWriter.Write(res);
        streamWriter.Close();
        streamWriter.Dispose();
        streamReader = new StreamReader(filePath + "/JsonData/" + path);
        ChooseType(type);
    }

    //选择存储数据类型
    private void ChooseType(int type)
    {
        string value = streamReader.ReadToEnd();
        switch (type)
        {
            case 1:
			FloorPlanData[] floorPlanData = JsonConvert.DeserializeObject<FloorPlanData[]>(value);
                foreach (var data in floorPlanData)
                {
                    FloorPlanDataDictionary.Add(data.rowNum, data);
                }

                break;
            case 2:
			SlideShowData[] slideShowDataData = JsonConvert.DeserializeObject<SlideShowData[]>(value);
                foreach (var data in slideShowDataData)
                {
                    SlideShowDataDictionary.Add(data.rowNum, data);
                }
                break;
            case 3:
			EightButtonData[] eightButtonData = JsonConvert.DeserializeObject<EightButtonData[]>(value);
                foreach (var data in eightButtonData)
                {
                    EightButtonDataDictionary.Add(data.rowNum, data);
                }
                break;
            case 4:
			RailData[] railData = JsonConvert.DeserializeObject<RailData[]>(value);
                foreach (var data in railData)
                {
                    RailDataDictionary.Add(data.rowNum, data);
                }
                break;
        }
        streamReader.Dispose();
    }

    //接收服务器资源包
    IEnumerator queren(int type, int classID, string path,int order)
    {
		res = null;
        WWWForm wf = new WWWForm();
        wf.AddField("ClassID", classID);
        wf.AddField("pageindex", 0);
        wf.AddField("pagesize", 999);
        wf.AddField("method", "shouye");
        WWW download = new WWW(serverPath + "/Areas/SchoolAgent/JZBSList.ashx", wf);
        yield return download;
        res = download.text;
        WriteDataByJson(type, path);
        ShowDataImage(order);
    }
    //加载数据
    #region Loading
    //加载房型图
    public void LoadFloorPlanDataByJson()
    {
        StartCoroutine(queren(1, 1, "/FloorPlanData.json",1));
    }
    //加载轮播图
    public void LoadSlideShowDataByJson()
    {
        StartCoroutine(queren(2, 10, "/SlideShowData.json",2));
    }

    //加载八个按钮
    public void LoadEightButtonDataByJson()
    {
        StartCoroutine(queren(3, 11, "/EightButtonData.json",3));
    }
    //加载横条
    public void LoadRailDataByJson()
    {
        StartCoroutine(queren(4, 12, "/RailData.json", 4));
    }

    #endregion

    //得到数据
    #region Get
    //得到房型图
    public FloorPlanData GetFloorPlanData(int id)
    {
        FloorPlanData floorPlanData; 
        if(FloorPlanDataDictionary.TryGetValue(id,out floorPlanData))
        {
            return floorPlanData;
        }
        Debug.LogError("房型图数据不存在");
        return null;
    }

    //获取轮播图
    public SlideShowData GetSlideShowData(int id)
    {
        SlideShowData slideShowData = new SlideShowData();
        if (SlideShowDataDictionary.TryGetValue(id, out slideShowData))
        {
            return slideShowData;
        }
        Debug.LogError("轮播图数据不存在");
        return null;
    }

    //获取八个按钮
    public EightButtonData GetEightButtonData(int id)
    {
        EightButtonData eightButtonData = new EightButtonData();
        if (EightButtonDataDictionary.TryGetValue(id, out eightButtonData))
        {
            return eightButtonData;
        }
        Debug.LogError("八个按钮数据不存在");
        return null;
    }

    //获取横条
    public RailData GetRailData(int id)
    {
        RailData railData = new RailData();
        if (RailDataDictionary.TryGetValue(id, out railData))
        {
            return railData;
        }
        Debug.LogError("横条数据不存在");
        return null;
    }
    #endregion
}

//嗯嗯嗯嗯嫩嗯嗯嗯讷讷嫩
