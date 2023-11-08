# recursive_method

```php
   public function addDescendents($prevLevel, $tid, &$ds, &$childDS){
        if( array_key_exists($tid, $childDS) ) {
            $level = $prevLevel + 1;
            foreach($childDS[$tid] as $childRow){
                $ds[] = array_merge($childRow, array('level'=>$level));
                $this->addDescendents($level, $childRow['tid'], $ds, $childDS);
            }
        }
        return;
    }

    public function getOptions( $data_type_2_key ){

        $result = db_query("SELECT ttd.vid, ttd.tid, ttd.name, ttd.weight,
                                IF(COALESCE(tth.parent, 0) > 0, 1, 0) AS hasParent,
                                COALESCE(tth.parent, 0) AS parentTid
                            FROM taxonomy_term_data ttd
                            LEFT JOIN taxonomy_term_hierarchy tth ON tth.tid = ttd.tid
                            WHERE ttd.vid IN (  SELECT vid
                                            FROM ghdx.taxonomy_vocabulary
                                            WHERE machine_name IN (" . $this->sections_csv . "))
                            AND ttd.tid IN (    SELECT entity_id
                                            FROM field_data_field_data_types_allowed
                                            WHERE field_data_types_allowed_tid = :tid  )
                            ORDER BY parentTid DESC, weight",
                            array(':tid' => $data_type_2_key));

        $dsUnsorted = $result->fetchAllAssoc('tid', PDO::FETCH_ASSOC);

        $ds = [];
        $childDS = [];
        foreach($dsUnsorted as $tid => $row){
            if($row['hasParent']){ // all these will be encountered first
                $childDS[ $row['parentTid'] ][] = $row;
                continue;
            }
            $ds[] = array_merge($row, array('level'=>0)); // top-level(0) parents
            $this->addDescendents(0, $tid, $ds, $childDS); // recursive
        }
        return $ds;
    }
```

Here is an example of a recursive method I wrote for a research organization.
Note the method getOptions calls the method addDescendents.
**The method addDescendents is a recursive method.**
You will notice that inside the method addDescendents is a call to addDescendents !
