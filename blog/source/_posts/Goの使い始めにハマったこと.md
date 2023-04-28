---
title: Goの使い始めにハマったこと
date: 2020-12-30 12:27:03
tags:
---

## GOPATH 関係
- GOのファイルはGOPATH以下のsrcディレクトリに格納しなければならない。  

- GoLandで自作のモジュールが補完でimportされないのは、GOPATH設定が正しくないから。
### 参考url
https://pleiades.io/help/go/configuring-goroot-and-gopath.html#gopath

## コードポイント 関係
ElasticSearchでqueryの結果が下記の`\u6771\u4eac0`ように日本語がUTF16のコードポイントとして表示されていた。
本当はprefの項目は東京0と表示されて欲しかった。
```
body, _ := ioutil.ReadAll(resp.Body)
mt.Println(string(body))

# コンソール出力
{"took":27,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":1,"relation":"eq"},"max_score":1.0,"hits":[{"_index":"search_job","_type":"_doc","_id":"0","_score":1.0,"_source":{"pref":"\u6771\u4eac0","employment":"full-time","line":{"name":"\u5c71\u624b\u7dda","station":["\u6771\u4eac"]}}}]}}
```

日本語がUTF16のコードポイントとして表示されており、
tranceformパッケージでurf16のバイトのDecodeを試したが文字化けして失敗。
(失敗例)
```
body, _ := ioutil.ReadAll(resp.Body)
hoge, _, _ := transform.Bytes(
    unicode.UTF16(unicode.LittleEndian, unicode.IgnoreBOM).NewDecoder(),
    body,
)
fmt.Println(string(hoge))

# コンソール出力
≻潴歯㨢ⰵ琢浩摥潟瑵㨢慦獬ⱥ弢桳牡獤㨢≻潴慴≬ㄺ∬畳捣獥晳汵㨢ⰱ猢敲慬楴湯㨢攢≱ⱽ洢硡獟潣敲㨢⸱ⰰ栢瑩≳嬺≻楟摮硥㨢猢慥捲彨潪≢∬瑟≦∺畜㜶ㄷ畜攴捡∰∬浥汰祯敭瑮㨢昢汵⵬楴敭Ⱒ氢湩≥笺渢浡≥∺畜挵ㄷ畜㈶戴畜搷慤Ⱒ猢慴楴湯㨢≛畜㜶ㄷ畜攴捡崢絽嵽絽
```

structにqueryの結果を入れて表示すると、期待通り東京0と表示されました。  
Unmarshalの中でdecodeがされているようです。
```
type GetResponse struct {
	Took     int  `json:"took"`
	TimedOut bool `json:"timed_out"`
	Shards   struct {
		Total      int `json:"total"`
		Successful int `json:"successful"`
		Skipped    int `json:"skipped"`
		Failed     int `json:"failed"`
	} `json:"_shards"`
	Hits struct {
		Total struct {
			Value    int    `json:"value"`
			Relation string `json:"relation"`
		} `json:"total"`
		MaxScore float64 `json:"max_score"`
		Hits     []struct {
			Index  string  `json:"_index"`
			Type   string  `json:"_type"`
			ID     string  `json:"_id"`
			Score  float64 `json:"_score"`
			Source struct {
				Pref       string `json:"pref"`
				Employment string `json:"employment"`
				Line       struct {
					Name    string   `json:"name"`
					Station []string `json:"station"`
				} `json:"line"`
			} `json:"_source"`
		} `json:"hits"`
	} `json:"hits"`
}

func main() {
    body, _ := ioutil.ReadAll(resp.Body)
	var getResponse GetResponse
	unmarshalErr := json.Unmarshal(body, &getResponse)
	if unmarshalErr != nil {
		fmt.Println(unmarshalErr)
	}

	for _, hit := range getResponse.Hits.Hits {
		fmt.Println("pref", hit.Source.Pref)
	}
}

# コンソール出力
pref 東京0
```

json全体ではなく、コードポイントの箇所のみであれば、Unquoteで文字に変換できました。
```
t := "\\u6771\\u4eac0"
converted, _ := strconv.Unquote(`"` + t + `"`)
fmt.Println(converted)
# コンソール出力
東京0
```
