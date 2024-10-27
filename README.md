![image](https://github.com/user-attachments/assets/70500c14-eae8-490c-af83-a6116b2c5af8)

# www.pipexy.com

## Protocol Streams

"Protocol" - directly references the gRPC/protocol foundation
"Streams" - evokes both: Data streaming/flow, Pipeline processing capabilities


## Przykłady pokazują różne scenariusze użycia:

1. Security Monitoring:
- Monitoring kamer
- Detekcja obiektów
- Kontrola dostępu

2. Production Monitoring:
- Kontrola jakości
- Monitoring maszyn
- Dane z czujników

3. Smart Home:
- Automatyzacja
- Kamery
- Czujniki IoT

4. Social Media Analytics:
- Analiza sentymentu
- Wykrywanie trendów
- Agregacja danych

5. Infrastructure Monitoring:
- Analiza logów
- Metryki sieciowe
- Alerty

Każdy pipeline pokazuje:
- Różne źródła danych
- Różne typy przetwarzania
- Różne formaty wyjściowe
- Różne systemy powiadomień

Można łatwo dostosować te przykłady przez:
- Zmianę parametrów
- Dodanie nowych tasków
- Modyfikację callbacków
- Dodanie nowych protokołów

```yaml
# config/pipelines.yaml

pipelines:
  # System monitoringu bezpieczeństwa
  - name: security_monitoring
    startup:
      - grpc://detector:50051/start?model=yolov5&confidence=0.6
      - grpc://face-detector:50052/start?model=face_recognition&min_size=80
    tasks:
      - input: rtsp://camera1.local:554/entrance?fps=15
        process: grpc://detector:50051/detect_objects?classes=person,vehicle
        callback: grpc://alert-service:50052/RegisterCallback

      - input: rtsp://camera2.local:554/parking?fps=10
        process: grpc://detector:50051/detect_objects?classes=car,truck,bicycle
        callback: grpc://alert-service:50052/RegisterCallback

      - input: rtsp://camera3.local:554/reception?fps=5
        process: grpc://face-detector:50052/recognize_faces
        callback: grpc://access-control:50053/RegisterCallback

    callback:
      "grpc://alert-service:50052/RegisterCallback":
        "grpc://localhost:50051/convertData":
          - file:///var/log/security/detections.json?mode=append
        "grpc://localhost:50051/convertDataForAlerts":
          - rss://security.local:8080/alerts?format=json&max_items=1000
          - webhook://slack.com/api/security-alerts?token=${SLACK_TOKEN}

  # System monitorowania produkcji
  - name: production_monitoring
    startup:
      - grpc://quality-detector:50055/start?model=defect_detection&threshold=0.8
      - grpc://metrics-collector:50056/start?interval=1s
    tasks:
      - input: rtsp://line1-camera/feed?fps=30
        process: grpc://quality-detector:50055/detect_defects
        callback: grpc://production-control:50057/QualityCallback

      - input: mqtt://sensors/temperature/+?interval=1s
        process: grpc://metrics-collector:50056/analyze_metrics
        callback: grpc://monitoring:50058/MetricsCallback

      - input: modbus://plc1/status?interval=100ms
        process: grpc://metrics-collector:50056/process_plc_data
        callback: grpc://monitoring:50058/PLCCallback

    callback:
      "grpc://production-control:50057/QualityCallback":
        "grpc://converter:50051/convertToMQTT":
          - mqtt://production/quality/line1?retain=true&qos=1
        "grpc://converter:50051/convertToDatabase":
          - postgresql://timescale:5432/metrics?table=quality_metrics

      "grpc://monitoring:50058/MetricsCallback":
        "grpc://converter:50051/convertToPrometheus":
          - prometheus://pushgateway:9091/metrics/job/production
        "grpc://converter:50051/convertToInflux":
          - influxdb://influx:8086/write?db=production&precision=ms

  # System IoT i smart home
  - name: smart_home_automation
    startup:
      - grpc://automation-engine:50060/start?config=home_rules
      - grpc://presence-detector:50061/start?sensitivity=high
    tasks:
      - input: mqtt://zigbee2mqtt/+/+/state?retain=true
        process: grpc://automation-engine:50060/process_state
        callback: grpc://home-control:50062/StateCallback

      - input: rtsp://doorbell-camera/stream?fps=10
        process: grpc://presence-detector:50061/detect_presence
        callback: grpc://notification:50063/PresenceCallback

      - input: mqtt://weather/outdoor/+?interval=5m
        process: grpc://automation-engine:50060/process_weather
        callback: grpc://home-control:50062/WeatherCallback

    callback:
      "grpc://home-control:50062/StateCallback":
        "grpc://converter:50051/convertToHomeAssistant":
          - mqtt://homeassistant/state/+?retain=true
        "grpc://converter:50051/convertToHistory":
          - influxdb://influx:8086/write?db=home_history

      "grpc://notification:50063/PresenceCallback":
        "grpc://converter:50051/convertToNotification":
          - pushover://user/notify?priority=high
          - telegram://bot/send?chat_id=${CHAT_ID}
        "grpc://converter:50051/convertToStorage":
          - s3://bucket/presence-events/

  # System analizy mediów społecznościowych
  - name: social_media_analytics
    startup:
      - grpc://sentiment-analyzer:50070/start?model=bert&language=multi
      - grpc://trend-detector:50071/start?interval=1m
    tasks:
      - input: twitter://api/stream?keywords=brand,product
        process: grpc://sentiment-analyzer:50070/analyze
        callback: grpc://analytics:50072/SentimentCallback

      - input: rss://news.feed/tech?interval=15m
        process: grpc://sentiment-analyzer:50070/analyze
        callback: grpc://analytics:50072/NewsCallback

      - input: websocket://reddit/stream?subreddits=technology,programming
        process: grpc://trend-detector:50071/detect_trends
        callback: grpc://analytics:50072/TrendCallback

    callback:
      "grpc://analytics:50072/SentimentCallback":
        "grpc://converter:50051/convertToElastic":
          - elasticsearch://elastic:9200/sentiment
        "grpc://converter:50051/convertToSlack":
          - webhook://slack/marketing?token=${SLACK_TOKEN}

      "grpc://analytics:50072/TrendCallback":
        "grpc://converter:50051/convertToVisualization":
          - grafana://dashboard/trends?key=${GRAFANA_KEY}
        "grpc://converter:50051/convertToEmail":
          - smtp://mail/send?to=team@company.com

  # System monitorowania infrastruktury
  - name: infrastructure_monitoring
    startup:
      - grpc://log-analyzer:50080/start?patterns=error,warning,critical
      - grpc://metric-collector:50081/start?interval=30s
    tasks:
      - input: syslog://servers/+/system?facility=*
        process: grpc://log-analyzer:50080/analyze_logs
        callback: grpc://monitoring:50082/LogCallback

      - input: snmp://network/+/metrics?community=public
        process: grpc://metric-collector:50081/collect_metrics
        callback: grpc://monitoring:50082/NetworkCallback

      - input: prometheus://scrape/targets/*?interval=15s
        process: grpc://metric-collector:50081/process_metrics
        callback: grpc://monitoring:50082/MetricsCallback

    callback:
      "grpc://monitoring:50082/LogCallback":
        "grpc://converter:50051/convertToELK":
          - elasticsearch://elastic:9200/logs
          - kibana://dashboard/system-logs
        "grpc://converter:50051/convertToPagerDuty":
          - pagerduty://api/event?severity=high

      "grpc://monitoring:50082/NetworkCallback":
        "grpc://converter:50051/convertToTimescale":
          - postgresql://timescale:5432/network_metrics
        "grpc://converter:50051/convertToGrafana":
          - grafana://dashboard/network?uid=network-overview
```


  

## Architecture Components:

1. Protocol Layer
- gRPC as communication backbone
- Protocol Buffers (protobuf) for schema definition
- Service contracts via .proto files
- Bi-directional streaming support
- Type-safe service interfaces

2. Stream Processing
```protobuf
service PipexyService {
  // Unary
  rpc TransformData (DataRequest) returns (DataResponse);
  
  // Server Streaming
  rpc StreamResults (QueryRequest) returns (stream ResultData);
  
  // Client Streaming
  rpc CollectMetrics (stream MetricData) returns (MetricsSummary);
  
  // Bi-directional Streaming
  rpc ProcessPipeline (stream PipelineStep) returns (stream PipelineResult);
}
```

3. DSL Integration
```scala
pipeline {
  source {
    grpc.stream("data.InputService/Subscribe")
      .withSchema("input.proto")
  }
  
  transform {
    filter(condition: "value > threshold")
    map(fields: ["timestamp", "metric", "value"])
    aggregate(window: "5m", fn: "avg")
  }
  
  sink {
    grpc.stream("metrics.OutputService/Publish")
      .withRetry(maxAttempts: 3)
  }
}
```

## Key Features

1. Protocol-Native
- Schema-first development
- Strong typing
- Contract-based APIs
- Backwards compatibility support

2. Stream Processing
- Real-time data handling
- Flow control (backpressure)
- Error handling & recovery
- Streaming patterns support:
  * Request-Response
  * Server Streaming
  * Client Streaming
  * Bi-directional Streaming

3. Pipeline Orchestration
- Declarative pipeline definition
- Stream composition
- Error handling strategies
- Monitoring & observability
- Resource management

## System Properties

1. Reliability
- Automatic reconnection
- Retry mechanisms
- Error recovery
- Transaction support

2. Performance
- Multiplexed connections
- Binary protocol efficiency
- Streaming optimization
- Resource pooling

3. Scalability
- Horizontal scaling
- Load balancing
- Partitioned streams
- Distributed processing

Example Usage Pattern:
```scala
// Define service contract
message DataStream {
  string stream_id = 1;
  bytes payload = 2;
  timestamp created_at = 3;
}

// Configure pipeline in DSL
val pipeline = Pipeline.define {
  // Input stream definition
  from("grpc://input-service:8080")
    .withProtocol("streams.proto")
    .asStream[DataStream]
    
  // Processing steps
  .transform { stream =>
    stream
      .filter(_.payload.size > 0)
      .map(enrichData)
      .window(TumblingWindow(5.minutes))
      .aggregate(computeMetrics)
  }
  
  // Output handling
  .to("grpc://metrics-service:8080")
    .withRetry(policy = exponentialBackoff)
    .withSchema("metrics.proto")
}
```


## The technical architecture combines

- Protocol-based communication (gRPC)
- Stream processing capabilities
- DSL for pipeline definition
- Type safety throughout
- Runtime flexibility
- Observable operations

