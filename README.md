# TEST

# 1-A번문제 풀이

    SELECT
        A.id,
        A.image_url,
        B.nickname,
        CASE WHEN C.id IS NULL THEN 'FALSE' ELSE 'TRUE' END AS is_scrap
    FROM cards A
             JOIN users B
                  ON A.user_id = B.id
             LEFT OUTER JOIN scraps C
                  ON A.id = C.card_id
    GROUP BY A.id
    ORDER BY A.id;

# 1-B번문제 풀이

    SELECT
        A.id,
        A.image_url,
        C.nickname,
        count(B.card_id) AS scrapper_count
    FROM cards A
             RIGHT OUTER JOIN scraps B
                              ON A.id = B.card_id
             JOIN users C
                  ON A.user_id = C.id
    GROUP BY B.card_id
    ORDER BY scrapper_count DESC;

# 1-C-1번문제 풀이 - 대표이미지 가장 처음 스크랩

    SELECT A.id,
           A.title,
           B.image_url,
           B.scrap_count
    FROM scrapbooks A
             JOIN (SELECT A.scrapbook_id,
                          B.image_url,
                          COUNT(c.card_id) as scrap_count
                   FROM(
                           SELECT scraps.*, ROW_NUMBER() OVER(PARTITION BY scrapbook_id ORDER BY created_at ASC) AS RN
                           FROM scraps
                       ) A
                           JOIN cards B ON A.card_id = B.id
                           JOIN scraps C ON A.scrapbook_id = C.scrapbook_id
                   WHERE RN = 1
                 GROUP BY A.scrapbook_id, B.image_url
                 ) B
                  ON A.id = B.scrapbook_id
    ORDER BY A.created_at;

# 1-C-2  - 대표이미지 가장 마지막 스크랩

    SELECT A.id,
           A.title,
           B.image_url,
           B.scrap_count
    FROM scrapbooks A
             JOIN (SELECT A.scrapbook_id,
                          B.image_url,
                          COUNT(c.card_id) as scrap_count
                   FROM(
                           SELECT scraps.*, ROW_NUMBER() OVER(PARTITION BY scrapbook_id ORDER BY created_at DESC) AS RN
                           FROM scraps
                       ) A
                           JOIN cards B ON A.card_id = B.id
                           JOIN scraps C ON A.scrapbook_id = C.scrapbook_id
                   WHERE RN = 1
                 GROUP BY A.scrapbook_id, B.image_url
                 ) B
                  ON A.id = B.scrapbook_id
    ORDER BY A.created_at;

# 1-C-3 - 가장 마지막 업로드 된 사진

    SELECT A.id,
           A.title,
           B.image_url,
           B.scrap_count
    FROM scrapbooks A
    JOIN (SELECT A.*,
                 COUNT(card_id) as scrap_count
          FROM(SELECT cards.*,
                      ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY created_at DESC) AS RN
               FROM cards) A
                    JOIN scraps B ON A.id = B.card_id
          where RN = 1
          GROUP BY card_id) B
    ON A.user_id = B.user_id
    GROUP BY A.id
    ORDER BY A.created_at;

# 2-A

    SELECT B.BRAND_NAME,
           SUM(A.COUNT) AS BUY_COUNT,
           SUM(A.COUNT * B.COST) AS BUY_AMOUNT
    FROM ORDERS A
             LEFT OUTER JOIN PRODUCTS B ON A.PRODUCT_ID = B.ID
    GROUP BY B.BRAND_NAME
    ORDER BY B.BRAND_NAME;

# 2-B

    SELECT A.NICKNAME,
           COUNT(*) AS BUY_COUNT,
           SUM(B.COUNT * C.COST) AS BUY_AMOUNT,
           CASE WHEN COUNT(*) >= 4 AND SUM(B.COUNT * C.COST) >= 1000000 THEN 'Platinum'
                WHEN COUNT(*) >= 3 AND SUM(B.COUNT * C.COST) >= 500000 THEN 'VIP'
                WHEN COUNT(*) >= 2 AND SUM(B.COUNT * C.COST) >= 300000 THEN 'Friend'
                ELSE 'Normal'
               END AS RATING
    FROM USERS A
             LEFT OUTER JOIN ORDERS B ON A.ID = B.USER_ID
             LEFT OUTER JOIN PRODUCTS C ON B.PRODUCT_ID = C.ID
    WHERE B.CREATE_AT > DATE_SUB(NOW(),INTERVAL 6 month)
    GROUP BY A.NICKNAME
    ORDER BY A.NICKNAME;

# 2-C
    SELECT SUM(A.COUNT) AS BUY_COUNT,
           SUM(A.COUNT * B.COST) AS BUY_AMOUNT
    FROM ORDERS A
             LEFT OUTER JOIN PRODUCTS B ON A.PRODUCT_ID = B.ID
    WHERE A.PRODUCT_ID IN
          (SELECT DISTINCT(PRODUCT_ID)
                FROM PRODUCT_CATEGORIES
           WHERE CATEGORY_ID = (SELECT ID FROM CATEGORIES WHERE FIRST = '가구' AND SECOND = '의자') -- 예시INPUT
          );

# 2-D
    SELECT B.name,
           SUM(A.COUNT) AS BUY_COUNT,
           SUM(A.COUNT * B.COST) AS BUY_AMOUNT
           FROM ORDERS A
       JOIN(SELECT * FROM (SELECT A.*,
                                  ROW_NUMBER() OVER(PARTITION BY category_id ORDER BY position ASC) AS RN
                                FROM product_categories A) A
                            JOIN products B ON A.product_id = B.id
                            WHERE RN = 1) B
        ON A.product_id = B.product_id
        GROUP BY A.product_id;
# 3-A
    //// -- LEVEL 1
    //// -- Tables and References

    // Creating tables
    //로그인테이블
    Table login as U {
      login_id int [pk, increment] // auto-increment
      nickname varchar(unique)
      email varchar(unique)
      password varchar
      created_at datetime
      updated_at datetime
    }

    //고객테이블
    Table customer as U {
      customer_id int [pk, increment] // auto-increment
      nickname varchar(fk) unique 
      gender varchar unique
      birthday datetime
      created_at datetime
      updated_at datetime
    }

    //사진테이블
    Table cards as U { 
      card_id int [pk, increment] // auto-increment
      nickname varchar(fk)
      category_id varchar(fk)
      image_url varchar
      created_at datetime
      updated_at datetime
    }

    //사진 카테고리 테이블
    Table card_category as U { 
      category_id int [pk, increment]
      category_desc text
      created_at datetime
      updated_at datetime
    }

    //스크랩 테이블
    Table scraps as U { 
      scrap_id int [pk, increment]
      nickname varchar(fk)
      scrap_title varchar
      created_at datetime
      updated_at datetime
    }

    //스크랩 타이틀별 사진구성 테이블
    Table scraps_title as U {
      scraps_title_id int [pk, increment]
      scrap_id int(fk) 
      image_layout varchar //이미지 구성 card_id 1|2|3|4|5
      created_at datetime
      updated_at datetime
    }

    Table card_comment as U { 
      cardcomment_id int [pk, increment]
      card_id int(fk) 
      nickname varchar //이미지 구성
      created_at datetime
      updated_at datetime
    }

    Table card_like as U { 
      cardLike_id int [pk, increment]
      card_id int(fk) 
      nickname varchar //이미지 구성
      created_at datetime
      updated_at datetime
    }

    Table scraps_comment as U { 
      scrapscomment_id int [pk, increment]
      scrap_id int(fk) 
      nickname varchar //이미지 구성
      created_at datetime
      updated_at datetime
    }

    Table scraps_like as U { 
      scraps_like_id int [pk, increment]
      scrap_id int(fk) 
      nickname varchar //이미지 구성
      created_at datetime
      updated_at datetime
    }

    Ref: "login"."nickname" - "customer"."nickname"

    Ref: "customer"."nickname" < "cards"."nickname"

    Ref: "login"."nickname" < "scraps"."nickname"

    Ref: "scraps"."scrap_id" < "scraps_title"."scrap_id"

    Ref: "cards"."category_id" < "card_category"."category_id"

    Ref: "scraps_title"."scrap_id" < "scraps_comment"."scrap_id"

    Ref: "scraps_comment"."scrap_id" < "scraps_like"."scrap_id"

    Ref: "cards"."card_id" < "card_comment"."card_id"

    Ref: "card_comment"."card_id" < "card_like"."card_id"
# 3-B

# 4-A

# 1번 풀이

       private void algorithm(){
          int[] a ={1,4,2};
          int[] b ={4,5,3};
          long reqTime = System.currentTimeMillis();
          Arrays.sort(a);
          Arrays.sort(b);

          int result = 0;
          for(int i=0; i<a.length; i++) {
             result += a[i] * b[b.length-i-1];
          }
          long resTime = System.currentTimeMillis();
          System.out.println("최소값 계산방식 알고리즘2 ="+result+"_"+ (resTime - reqTime)/1000.000);
       }
   
# 2번 풀이

       private void algorithm() {
          Integer[] a = {1,4,2};
          Integer[] b = {4,5,3};
          long reqTime = System.currentTimeMillis();
          Arrays.sort(a);
          Arrays.sort(b,Comparator.reverseOrder());

          int result = 0 ;
          for(int i =0 ; i < a.length; i++){
             result += a[i] * b[i];
          }
          long resTime = System.currentTimeMillis();
          System.out.println("최소값 Wrapper Class 함수형 리버스="+ result +"_"+ (resTime - reqTime)/1000.000 );
       }
   
# 4-B   

       private void algorithm(){
          Integer[] frontTeam = {10,8,2,14};
          Integer[] backEndTeam = {4,4,12,16};
          int winCount = 0;

          for(int i=0; i < frontTeam.length; i++){
             List<Integer> players = new ArrayList<>(); //이길수 있는 백엔드 팀 플레이어 수
             for(int j=0; j < backEndTeam.length; j++){ //프론트 팀과 백엔드 팀 참가자 수는 같음.
                if(frontTeam[i] < backEndTeam[j]){
                   players.add(backEndTeam[j]);
                }
             }
             if(players.size() > 0) {
                Collections.sort(players);
                ArrayUtils.remove(backEndTeam,players.get(0));
                winCount++;
             }
          }
       }
