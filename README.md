# SQL_CTE
## Scheme and Creation for a SWAPS Programs 

```-- Create schema with UTF-8 support
CREATE SCHEMA IF NOT EXISTS swaps_analytics
  DEFAULT CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

USE swaps_analytics;

-- Jobcentre dimension (no PII)
CREATE TABLE IF NOT EXISTS dim_jobcentre (
  jobcentre_key      INT AUTO_INCREMENT PRIMARY KEY,
  jobcentre_code     VARCHAR(20)  NOT NULL UNIQUE,
  jobcentre_name     VARCHAR(100) NOT NULL,
  district           VARCHAR(100),
  region             VARCHAR(100),
  site_type          VARCHAR(30),             -- main / outreach
  opening_date       DATE,
  closure_date       DATE DEFAULT NULL,       -- Nullable for legacy sites
  effective_from     DATE NOT NULL,
  effective_to       DATE DEFAULT NULL,
  is_current         TINYINT(1) NOT NULL DEFAULT 1
) ENGINE=InnoDB;

-- Provider dimension
CREATE TABLE IF NOT EXISTS dim_provider (
  provider_key   INT AUTO_INCREMENT PRIMARY KEY,
  provider_code  VARCHAR(30) NOT NULL UNIQUE,
  provider_name  VARCHAR(150) NOT NULL,
  region         VARCHAR(100),
  active_flag    TINYINT(1) NOT NULL DEFAULT 1
) ENGINE=InnoDB;

-- Course dimension
CREATE TABLE IF NOT EXISTS dim_course (
  course_key        INT AUTO_INCREMENT PRIMARY KEY,
  course_code       VARCHAR(40) NOT NULL UNIQUE,
  course_name       VARCHAR(200) NOT NULL,
  sector            VARCHAR(100),             -- e.g., Warehousing, Care, Digital
  format_type       VARCHAR(30),              -- live / hybrid / self-paced
  content_style     VARCHAR(30),              -- interactive / graphics-rich / static
  duration_days     DECIMAL(5,2),
  level_band        VARCHAR(30),              -- entry / intermediate / advanced
  active_flag       TINYINT(1) NOT NULL DEFAULT 1
) ENGINE=InnoDB;

-- Interest taxonomy
CREATE TABLE IF NOT EXISTS dim_interest (
  interest_key    INT AUTO_INCREMENT PRIMARY KEY,
  interest_code   VARCHAR(40) NOT NULL UNIQUE,
  interest_name   VARCHAR(120) NOT NULL
) ENGINE=InnoDB;

-- No-show / withdrawal reasons
CREATE TABLE IF NOT EXISTS dim_reason (
  reason_key     INT AUTO_INCREMENT PRIMARY KEY,
  reason_code    VARCHAR(40) NOT NULL UNIQUE,
  reason_group   VARCHAR(60),                 -- transport, caring, health, mismatch, admin, unknown
  reason_label   VARCHAR(200) NOT NULL
) ENGINE=InnoDB;

-- Date dimension
CREATE TABLE IF NOT EXISTS dim_date (
  date_key     INT PRIMARY KEY,               -- YYYYMMDD
  cal_date     DATE NOT NULL,
  cal_year     INT,
  cal_qtr      TINYINT,
  cal_month    TINYINT,
  cal_week     TINYINT
) ENGINE=InnoDB;```


### 🧩 SQL INNER JOIN

```sql
SELECT 
    p.provider_name,
    p.region AS provider_region,
    c.course_name,
    c.sector,
    c.format_type,
    c.duration_days
FROM 
    dim_provider p
INNER JOIN 
    dim_course c
    ON p.provider_code = c.course_code  -- Replace with actual relationship key
WHERE 
    p.active_flag = 1 AND c.active_flag = 1
ORDER BY 
    p.provider_name, c.course_name;
```

---

### ✅ Notes:
- Replace `p.provider_code = c.course_code` with the correct foreign key if you have a bridge or fact table (e.g., `fact_course_provider`).
- This query filters for **active** providers and courses.
- You can easily extend this to include metrics from a fact table like attendance, referrals, or outcomes.

Would you like help designing a proper fact table to link providers and courses more robustly? I can sketch out a schema that supports cohort tracking, referral sources, and outcome metrics.
