# install pyhton3

apk add python3

# create a ds enviroment

python3 -m venv piscine

# activate the virtual environment

source piscine/bin/activate

# install required packages
pip install "psycopg[binary]"


# Check unique values 

```sh
cat  data_202?_???.csv | cut -d, -f2 | sort | uniq
```

|event_type      |
|----------------|
|cart            |
|purchase        |
|remove_from_cart|
|view            |

cat  data_202?_???.csv | cut -d, -f3 | sort -n| uniq | sed -
n '1p;2p;$p'
|product_id| |
|----------|-|
|3752|min|
|5924514|max|


cat  data_202?_???.csv | cut -d, -f4 | sort -n| uniq | sed -n '1p;2p;$p'

|price| |
|----------|-|
|-79.37|min
|327.78|max|


cat  data_202?_???.csv | cut -d, -f4 | sort -n| uniq | sed -
n '1p;2p;$p'

|user_id| |
|----------|-|
|465496|min
|608822072|max|


Both columns, user_id and product_id,  currently fit in INTEGER, whose maximum is 2,147,483,647. But look at how much of that range each one already uses: product_id tops out at about 5.9 million, which is roughly 0.3% of the range, so INTEGER is plenty. user_id, on the other hand, already reaches about 609 million, which is around 28% of the range. User IDs tend to grow faster than product IDs, so moving it to BIGINT (maximum around 9.2 quintillion) removes any risk of overflow later, at the cost of 4 extra bytes per row.
