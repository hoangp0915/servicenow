# Copy column table
```
(function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {
    // Get query params
    var queryParams = request.queryParams;
    var target = queryParams.target_table[0];
    var source = queryParams.source_table[0];

    // Delete all column of target table
    deleteAllColumnTargetTable();

    // Build query
    var query = 'name=' + source;
	findExtendTable(source)

    // insert column table
    var gr = new GlideRecord('sys_dictionary');
    gr.addEncodedQuery(query + '^elementISNOTEMPTY^elementNOT LIKEsys_');
    gr.query();
    gs.info('[QUERY] ' + gr.getEncodedQuery());
    gs.info('[START] Create column for target table');
    while (gr.next()) {
        insertColumn(gr);
    }
    gs.info('[END] Create column for target table');

    function insertColumn(gr) {
        try {
            var grhw = new GlideRecord('sys_dictionary');
            grhw.initialize();
            grhw.name = target;
            grhw.internal_type = gr.internal_type;
            grhw.default_value = gr.default_value;
            grhw.column_label = gr.column_label;
            grhw.element = gr.element;
            grhw.mandatory = gr.mandatory;
            grhw.reference = gr.reference;
            grhw.max_length = gr.max_length;
            grhw.read_only = true;
            grhw.active = gr.active;
            grhw.default_value = gr.default_value;
            grhw.insert();
        } catch (ex) {
            gs.error('Error: ' + gr.column_label);
        }
    }

    function deleteAllColumnTargetTable() {
        var grTb = new GlideRecord('sys_dictionary');
        grTb.addEncodedQuery('name=' + target + '^elementISNOTEMPTY^elementNOT LIKEsys_');
        grTb.query();
        if (grTb.hasNext()) {
            gs.info('[START] Delete all column');
            grTb.deleteMultiple();
            gs.info('[END] Delete all column');
        }
    }

    function findExtendTable(table) {
        var gr = new GlideRecord('sys_db_object');
        gr.addQuery('name', table);
        gr.query();
		gs.info('getEncodedQuery' + gr.getEncodedQuery());
        if (gr.next()) {
            gs.info('gr.super_class' + gr.super_class.name);
            if (gr.super_class) {
                query += '^ORname=' + gr.super_class.name;
                findExtendTable(gr.super_class.name);
            }
        }
    }
    response.setStatus(200);
    response.setBody({
        message: "Success"
    });
})(request, response);
```
# Copy choice
```
(function process(/*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {

    // Get query params
    var queryParams = request.queryParams;
    var target = queryParams.target_table[0];
    var source = queryParams.source_table[0];
	
	deleteAllChoice(target);

	// Build query
    var query = 'name=' + source;
    findExtendTable(source);
	
	// insert column table
    var gr = new GlideRecord('sys_dictionary');
    gr.addEncodedQuery(query + '^elementISNOTEMPTY^elementNOT LIKEsys_');
    gr.query();

    gs.info('[START] Insert choice for target table');
    while (gr.next()) {
        insertChoice(gr, query);
    }
    gs.info('[END] Insert choice for target table');
	
	function findExtendTable(table) {
        var gr = new GlideRecord('sys_db_object');
        gr.addQuery('name', table);
        gr.query();
        if (gr.next()) {
            if (gr.super_class) {
                query += '^ORname=' + gr.super_class.name;
                findExtendTable(gr.super_class.name);
            }
        }
    }
	
	function insertChoice(glideRecord, queryTable) {
        var element = glideRecord.getValue('element');
        var gr = new GlideRecord('sys_choice');
        gr.addEncodedQuery('element=' + element + '^' + queryTable);
		gs.info('[QUERY] '+gr.getEncodedQuery());
        gr.query();
        while (gr.next()) {
            var ins = new GlideRecord('sys_choice');
            ins.initialize();
            ins.name = target;
            ins.element = gr.element;
            ins.language = gr.language;
            ins.sequence = gr.sequence;
            ins.value = gr.value;
            ins.label = gr.label;
            ins.inactive = gr.inactive;
			gs.info(JSON.stringify(ins.sys_meta));
            ins.insert();
        }
    }
	
	function deleteAllChoice(table) {
        var gr = new GlideRecord('sys_choice');
        gr.addEncodedQuery('name=' + table);
        gr.query();
        if (gr.next()) {
            gr.deleteMultiple();
        }
    }
	response.setStatus(200);
	response.setBody({message: "Success"});
})(request, response);
```
