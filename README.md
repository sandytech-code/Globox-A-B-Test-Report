# Globox-A-B-Test-Report
WITH user_counts AS (
    SELECT
        g.join_dt AS date,
        g.group AS group_id,
        COUNT(DISTINCT u.id) AS user_count
    FROM
        groups g
    INNER JOIN
        users u ON g.uid = u.id
    GROUP BY
        g.join_dt, g.group
),
converted_user_counts AS (
    SELECT
        a.dt AS date,
        g.group AS group_id,
        COUNT(DISTINCT g.uid) AS converted_user_count
    FROM
        activity a
    INNER JOIN
        groups g ON a.uid = g.uid
    GROUP BY
        a.dt, g.group
),
cumulative_counts AS (
    SELECT
        uc.date,
        uc.group_id,
        uc.user_count,
        SUM(uc.user_count) OVER (PARTITION BY uc.group_id ORDER BY uc.date) AS cum_users,
        cu.converted_user_count,
        SUM(cu.converted_user_count) OVER (PARTITION BY cu.group_id ORDER BY cu.date) AS cum_converted_users
    FROM
        user_counts uc
    LEFT JOIN
        converted_user_counts cu ON uc.date = cu.date AND uc.group_id = cu.group_id
)
SELECT
    cc.date,
    AVG(CASE WHEN cc.group_id = 'A' THEN cc.cum_converted_users::numeric / cc.cum_users END) AS conversion_rate_A,
    AVG(CASE WHEN cc.group_id = 'B' THEN cc.cum_converted_users::numeric / cc.cum_users END) AS conversion_rate_B,
    AVG(CASE WHEN cc.group_id = 'B' THEN cc.cum_converted_users::numeric / cc.cum_users END) - AVG(CASE WHEN cc.group_id = 'A' THEN cc.cum_converted_users::numeric / cc.cum_users END) AS difference
FROM
    cumulative_counts cc
GROUP BY
    cc.date
ORDER BY
    Cc.date;
