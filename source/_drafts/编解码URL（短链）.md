---
title: 编解码URL（短链）
tags:
---



### Stack+StringBuilder
      public class Codec {
          Map<String, String> url2Code = new HashMap<String, String>();
          Map<String, String> code2Url = new HashMap<String, String>();
          char[] baseChars = new char[]{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n'};

          public String encode(String longUrl) {
              if (longUrl == null) return null;
              if (url2Code.containsKey(longUrl)) return url2Code.get(longUrl);

              StringBuilder stringBuilder = new StringBuilder();
              String code;
              int length = baseChars.length;

              do {
                  for (int i = 0; i < 6; i++) {
                      stringBuilder.append(baseChars[(int) Math.random() * length]);
                  }
                  code = stringBuilder.toString();
              } while (url2Code.containsKey(code));

              if (longUrl.startsWith("http:")) {
                  code= "http:xx.com/" + code;
              } else {
                  code= "https:xx.com/" + code;
              }
              url2Code.put(longUrl, code);
              code2Url.put(code, longUrl);
              return code;
          }

          public String decode(String shortUrl) {
              return code2Url.get(shortUrl);
          }
      }

      // Your Codec object will be instantiated and called as such:
      // Codec codec = new Codec();
      // codec.decode(codec.encode(url));




     public class Codec {
             HashMap<String, String> urlToIndexMap = new HashMap<String, String>();
              HashMap<String, String> indexToUrlMap = new HashMap<String, String>();

             public int i;

             public String encode(String longUrl) {
                 urlToIndexMap.put(longUrl, i+"");
                 indexToUrlMap.put(i+"", longUrl);
                 return i+"";
             }

             public String decode(String shortUrl) {
                 return "" + indexToUrlMap.get(shortUrl);
             }
     }

     // Your Codec object will be instantiated and called as such:
     // Codec codec = new Codec();
     // codec.decode(codec.encode(url));
