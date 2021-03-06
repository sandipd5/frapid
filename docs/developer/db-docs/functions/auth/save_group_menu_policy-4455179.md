# auth.save_group_menu_policy function:

```plpgsql
CREATE OR REPLACE FUNCTION auth.save_group_menu_policy(_role_id integer, _office_id integer, _menus text, _app_name text)
RETURNS void
```
* Schema : [auth](../../schemas/auth.md)
* Function Name : save_group_menu_policy
* Arguments : _role_id integer, _office_id integer, _menus text, _app_name text
* Owner : frapid_db_user
* Result Type : void
* Description : 


**Source:**
```sql
CREATE OR REPLACE FUNCTION auth.save_group_menu_policy(_role_id integer, _office_id integer, _menus text, _app_name text)
 RETURNS void
 LANGUAGE plpgsql
AS $function$
    DECLARE _menu_ids      integer[] = public.text_to_int_array(_menus);
BEGIN
    IF(_role_id IS NULL OR _office_id IS NULL) THEN
        RETURN;
    END IF;
    
    DELETE FROM auth.group_menu_access_policy
    WHERE auth.group_menu_access_policy.menu_id NOT IN(SELECT * from unnest(_menu_ids))
    AND role_id = _role_id
    AND office_id = _office_id
    AND menu_id IN
    (
        SELECT menu_id
        FROM core.menus
        WHERE _app_name = ''
        OR app_name = _app_name
    );

    WITH menus
    AS
    (
        SELECT unnest(_menu_ids) AS _menu_id
    )
    
    INSERT INTO auth.group_menu_access_policy(role_id, office_id, menu_id)
    SELECT _role_id, _office_id, _menu_id
    FROM menus
    WHERE _menu_id NOT IN
    (
        SELECT menu_id
        FROM auth.group_menu_access_policy
        WHERE auth.group_menu_access_policy.role_id = _role_id
        AND auth.group_menu_access_policy.office_id = _office_id
    );

    RETURN;
END
$function$

```

### Related Contents
* [Schema List](../../schemas.md)
* [Function List](../../functions.md)
* [Trigger List](../../triggers.md)
* [Table of Contents](../../README.md)

