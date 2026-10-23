# Load Test Results: 6 Flask Servers (`hey`)

## Setup

| | |
|---|---|
| Date | Friday, Oct 23, 2026 |
| Flask servers | 6 (t3.micro, gunicorn `--workers 2`) |
| Proxy | nginx on 1 t3.micro, round-robin |
| Database | DynamoDB Local on 1 t3.micro |
| Load tool | `hey`, run from a MacBook |
| URL | `http://100.53.178.74/` |
| Requests per run | 2000 (`-n 2000`) |
| Concurrency | 20 to 200 in steps of 20 |

Command:

```bash
for c in 20 40 60 80 100 120 140 160 180 200; do
  hey -n 2000 -c $c http://100.53.178.74/
done
```

## Summary

| Concurrency | Requests/sec | Average (ms) | Fastest (ms) | Slowest (ms) | Responses (all 200) |
|---|---|---|---|---|---|
| 20 | 727 | 27.3 | 17.4 | 114.6 | 2000 |
| 40 | 1,414 | 28.1 | 17.5 | 120.5 | 2000 |
| 60 | 1,805 | 32.8 | 18.7 | 111.9 | 1980 |
| 80 | 1,770 | 43.3 | 22.5 | 145.1 | 2000 |
| 100 | 2,705 | 36.1 | 20.0 | 110.9 | 2000 |
| 120 | 2,497 | 41.9 | 20.9 | 131.6 | 1920 |
| 140 | 3,029 | 41.6 | 20.3 | 151.9 | 1960 |
| 160 | 3,073 | 45.4 | 22.7 | 149.2 | 1920 |
| 180 | 3,145 | 48.2 | 24.5 | 157.7 | 1980 |
| 200 | 4,057 | 46.3 | 25.5 | 150.8 | 2000 |

## Observations

- Throughput kept climbing all the way to c=200, and average latency only rose from about 27 ms to about 46 ms. Six Flask servers did not reach a ceiling in this range.
- Every response was HTTP 200. The counts below 2000 are not failures: `hey` rounds `-n` down to a multiple of `-c`, so for example `-n 2000 -c 60` sends 1980 requests.
- The dips at c=80 and c=120 are likely noise. Each run lasts under a second, so small hiccups have a large effect.
- `/` returns a static HTML page and never reads from the database. CPU sampled during a c=200 run showed the proxy about 80% idle, the Flask servers about 87% idle, and the database 100% idle.
- `hey` reuses connections (keep-alive) by default, so its numbers are higher than `ab`'s, which opens a new connection per request. The two tools should not be compared directly.

## Raw Output

### c=20

```text
Slowest:      0.1146 secs
Fastest:      0.0174 secs
Average:      0.0273 secs
Requests/sec: 727.3426

0.017 [1]     |
0.027 [1782]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.037 [97]    |■■
0.047 [0]     |
0.056 [20]    |
0.066 [0]     |
0.076 [0]     |
0.085 [0]     |
0.095 [17]    |
0.105 [56]    |■
0.115 [27]    |■

[200] 2000 responses
```

### c=40

```text
Slowest:      0.1205 secs
Fastest:      0.0175 secs
Average:      0.0281 secs
Requests/sec: 1414.0194

0.017 [1]     |
0.028 [1730]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.038 [149]   |■■■
0.048 [40]    |■
0.059 [0]     |
0.069 [0]     |
0.079 [0]     |
0.090 [0]     |
0.100 [0]     |
0.110 [50]    |■
0.121 [30]    |■

[200] 2000 responses
```

### c=60

```text
Slowest:      0.1119 secs
Fastest:      0.0187 secs
Average:      0.0328 secs
Requests/sec: 1804.6720

0.019 [1]     |
0.028 [1227]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.037 [507]   |■■■■■■■■■■■■■■■■■
0.047 [60]    |■■
0.056 [8]     |
0.065 [57]    |■■
0.075 [0]     |
0.084 [0]     |
0.093 [0]     |
0.103 [53]    |■■
0.112 [67]    |■■

[200] 1980 responses
```

### c=80

```text
Slowest:      0.1451 secs
Fastest:      0.0225 secs
Average:      0.0433 secs
Requests/sec: 1769.8053

0.023 [1]     |
0.035 [1136]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.047 [483]   |■■■■■■■■■■■■■■■■■
0.059 [96]    |■■■
0.072 [10]    |
0.084 [43]    |■■
0.096 [37]    |■
0.108 [54]    |■■
0.121 [84]    |■■■
0.133 [31]    |■
0.145 [25]    |■

[200] 2000 responses
```

### c=100

```text
Slowest:      0.1109 secs
Fastest:      0.0200 secs
Average:      0.0361 secs
Requests/sec: 2705.4751

0.020 [1]     |
0.029 [584]   |■■■■■■■■■■■■■■■■■■■■■
0.038 [1114]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.047 [109]   |■■■■
0.056 [69]    |■■
0.065 [23]    |■
0.075 [0]     |
0.084 [0]     |
0.093 [0]     |
0.102 [5]     |
0.111 [95]    |■■■

[200] 2000 responses
```

### c=120

```text
Slowest:      0.1316 secs
Fastest:      0.0209 secs
Average:      0.0419 secs
Requests/sec: 2496.7592

0.021 [1]     |
0.032 [640]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.043 [850]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.054 [199]   |■■■■■■■■■
0.065 [74]    |■■■
0.076 [17]    |■
0.087 [0]     |
0.098 [0]     |
0.109 [1]     |
0.120 [114]   |■■■■■
0.132 [24]    |■

[200] 1920 responses
```

### c=140

```text
Slowest:      0.1519 secs
Fastest:      0.0203 secs
Average:      0.0416 secs
Requests/sec: 3029.4685

0.020 [1]     |
0.033 [1146]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.047 [598]   |■■■■■■■■■■■■■■■■■■■■■
0.060 [0]     |
0.073 [0]     |
0.086 [0]     |
0.099 [9]     |
0.112 [66]    |■■
0.126 [15]    |■
0.139 [102]   |■■■■
0.152 [23]    |■

[200] 1960 responses
```

### c=160

```text
Slowest:      0.1492 secs
Fastest:      0.0227 secs
Average:      0.0454 secs
Requests/sec: 3072.9686

0.023 [1]     |
0.035 [1130]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.048 [526]   |■■■■■■■■■■■■■■■■■■■
0.061 [51]    |■■
0.073 [0]     |
0.086 [0]     |
0.099 [0]     |
0.111 [1]     |
0.124 [30]    |■
0.137 [104]   |■■■■
0.149 [77]    |■■■

[200] 1920 responses
```

### c=180

```text
Slowest:      0.1577 secs
Fastest:      0.0245 secs
Average:      0.0482 secs
Requests/sec: 3145.3253

0.024 [1]     |
0.038 [969]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.051 [732]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.064 [85]    |■■■■
0.078 [0]     |
0.091 [0]     |
0.104 [4]     |
0.118 [4]     |
0.131 [16]    |■
0.144 [97]    |■■■■
0.158 [72]    |■■■

[200] 1980 responses
```

### c=200

```text
Slowest:      0.1508 secs
Fastest:      0.0255 secs
Average:      0.0463 secs
Requests/sec: 4057.1950

0.025 [1]     |
0.038 [1177]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
0.051 [621]   |■■■■■■■■■■■■■■■■■■■■■
0.063 [1]     |
0.076 [0]     |
0.088 [0]     |
0.101 [0]     |
0.113 [5]     |
0.126 [32]    |■
0.138 [77]    |■■■
0.151 [86]    |■■■

[200] 2000 responses
```
