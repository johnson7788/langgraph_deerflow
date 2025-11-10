# 主要的对话接口，src/server/app.py中的函数@app.post("/api/chat/stream")
```
curl 'http://localhost:9000/api/chat/stream' \
  -H 'sec-ch-ua-platform: "macOS"' \
  -H 'Cache-Control: no-cache' \
  -H 'Referer: http://localhost:3000/' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36' \
  -H 'sec-ch-ua: "Chromium";v="134", "Not:A-Brand";v="24", "Google Chrome";v="134"' \
  -H 'Content-Type: application/json' \
  -H 'sec-ch-ua-mobile: ?0' \
  --data-raw '{"messages":[{"role":"user","content":"你好"}],"thread_id":"5wj_XJHWvlE_R_b8DF1jG","resources":[],"auto_accepted_plan":false,"enable_deep_thinking":false,"enable_background_investigation":false,"max_plan_iterations":1,"max_step_num":3,"max_search_results":3,"report_style":"social_media"}'

```

# 提示词增强接口，src/server/app.py中的函数@app.post("/api/prompt/enhance")
curl 'http://localhost:9000/api/prompt/enhance' \
  -H 'sec-ch-ua-platform: "macOS"' \
  -H 'Referer: http://localhost:3000/' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36' \
  -H 'sec-ch-ua: "Chromium";v="134", "Not:A-Brand";v="24", "Google Chrome";v="134"' \
  -H 'Content-Type: application/json' \
  -H 'sec-ch-ua-mobile: ?0' \
  --data-raw '{"prompt":"你好","report_style":"SOCIAL_MEDIA"}'