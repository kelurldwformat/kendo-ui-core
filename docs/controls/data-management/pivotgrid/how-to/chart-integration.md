---
title: Integrate with Kendo UI Chart
page_title: Integrate with Kendo UI Chart | Kendo UI PivotGrid
description: "Learn how to integrate a Kendo UI PivotGrid widget with a Kendo UI Chart widget."
slug: howto_integratewith_kendoui_chart_pivotgrid
---

# Integrate with Kendo UI Chart

The example below demonstrates how to integrate a Kendo UI PivotGrid widget with a Kendo UI Chart widget.

> **Important**
>
> The example omits the totals from the PivotGrid, so they are not shown in the Chart data visualization.

###### Example

```html
<div id="example">
    <div id="pivotgrid"></div>
    <div id="chart"></div>
    <script>
        //function flatten the tree of tuples that datasource returns
        function flattenTree(tuples) {
            tuples = tuples.slice();
            var result = [];
            var tuple = tuples.shift();
            var idx, length, spliceIndex, children, member;

            while (tuple) {
                //required for multiple measures
                if (tuple.dataIndex !== undefined) {
                    result.push(tuple);
                }

                spliceIndex = 0;
                for (idx = 0, length = tuple.members.length; idx < length; idx++) {
                    member = tuple.members[idx];
                    children = member.children;
                    if (member.measure) {
                        [].splice.apply(tuples, [0, 0].concat(children));
                    } else {
                        [].splice.apply(tuples, [spliceIndex, 0].concat(children));
                    }
                    spliceIndex += children.length;
                }

                tuple = tuples.shift();
            }

            return result;
        }

        function fullPath(members, idx) {
            var path = [];
          var parentName = members[idx].parentName;

          idx -= 1;

          while (idx > -1) {
            path.push(members[idx].name);
            idx -= 1;
          }

          path.push(parentName);

          return path;
        }

        //Check whether the tuple has been collapsed
        function isCollapsed(tuple, collapsed) {
          var members = tuple.members;

          for (var idx = 0, length = collapsed.length; idx < length; idx++) {
            var collapsedPath = collapsed[idx];
            if (indexOfPath(fullPath(members, collapsedPath.length -1), [collapsedPath]) !== -1) {
              return true;
            }
          }

          return false;
        }

        function indexOfPath(path, paths) {
            var path = path.join(",");

          for (var idx = 0; idx < paths.length; idx++) {
            if (paths[idx].join(",") === path) {
                return idx;
            }
          }

          return -1;
        }

        function fullPathCaptionName(members, idx) {
            var name = "";
            while (idx > -1) {
                            name += " " + members[idx].caption;
              idx -= 1;
            }
            return name;
        }

        //the main function that convert PivotDataSource data into understandable for the Chart widget format
        function convertData(dataSource, collapsed) {
          var columnTuples = flattenTree(dataSource.axes().columns.tuples || [], collapsed.columns);
          var rowTuples = flattenTree(dataSource.axes().rows.tuples || [], collapsed.rows);
          var data = dataSource.data();
          var rowTuple, columnTuple;

          var idx = 0;
          var result = [];
          var columnsLength = columnTuples.length;

          for (var i = 0; i < rowTuples.length; i++) {
            rowTuple = rowTuples[i];

            if (!isCollapsed(rowTuple, collapsed.rows)) {
              for (var j = 0; j < columnsLength; j++) {
                columnTuple = columnTuples[j];

                if (!isCollapsed(columnTuple, collapsed.columns)) {
                  if (idx > columnsLength && idx % columnsLength !== 0) {
                    for (var ri = 0; ri < rowTuple.members.length; ri++) {
                      for (var ci = 0; ci < columnTuple.members.length; ci++) {
                        //do not add root tuple members, e.g. [CY 2005, All Employees]
                        //Add only children
                        if (!columnTuple.members[ci].parentName || !rowTuple.members[ri].parentName) {
                          continue;
                        }

                        result.push({
                          measure: Number(data[idx].value),
                          column: fullPathCaptionName(columnTuple.members, ci),
                          row: rowTuple.members[ri].caption
                        });
                      }
                    }
                  }
                }
                idx += 1;
              }
            }
          }

          return result;
        }
    </script>
    <script>
        $(document).ready(function () {
            var collapsed = {
              columns: [],
              rows: []
            };

            var pivotgrid = $("#pivotgrid").kendoPivotGrid({
                filterable: true,
                //gather the collapsed members
                collapseMember: function(e) {
                  var axis = collapsed[e.axis];
                  var path = e.path;

                  if (indexOfPath(path, axis) === -1) {
                     axis.push(path);
                  }
                },
                //gather the expanded members
                expandMember: function(e) {
                  var axis = collapsed[e.axis];
                  var index = indexOfPath(e.path, axis);

                  if (index !== -1) {
                    axis.splice(index, 1);
                  }
                },
                columnWidth: 100,
                height: 330,
                dataSource: {
                    type: "xmla",
                    columns: [{ name: "[Date].[Calendar]", expand: true }, { name: "[Employee].[Gender]" }],
                  rows: [{ name: "[Product].[Category]", expand: true }],
                    measures: ["[Measures].[Reseller Freight Cost]"],
                    transport: {
                        connection: {
                            catalog: "Adventure Works DW 2008R2",
                            cube: "Adventure Works"
                        },
                        read: "http://demos.telerik.com/olap/msmdpump.dll"
                    },
                    schema: {
                        type: "xmla"
                    },
                    error: function (e) {
                        alert("error: " + kendo.stringify(e.errors[0]));
                    }
                },
                dataBound: function() {
                  //create/bind the chart widget
                  initChart(convertData(this.dataSource, collapsed));
                }
            }).data("kendoPivotGrid");

            function initChart(data) {
              var chart = $("#chart").data("kendoChart");

              if (!chart) {
                $("#chart").kendoChart({
                  dataSource: {
                    data: data,
                    group: "row"
                  },
                  series: [{
                      type: "column",
                      field: "measure",
                      name: "#= group.value # (category)",
                      categoryField: "column"
                  }],
                  legend: {
                      position: "bottom"
                  },
                  valueAxis: {
                      labels: {
                          format: "${0}"
                      }
                  },
                  dataBound: function(e) {
                    if (e.sender.options.categoryAxis) {
                        e.sender.options.categoryAxis.categories.sort()
                    }
                  }
                });
              } else {
                chart.dataSource.data(data);
              }
            }
        });
    </script>
</div>
```

## See Also

Other articles and how-to examples on the Kendo UI PivotGrid:

* [PivotGrid JavaScript API Reference](/api/javascript/ui/pivotgrid)
* [How to Change Data Source Dynamically]({% slug howto_change_datasource_dynamically_pivotgrid %})
* [How to Drill Down Navigation Always Starting from Root Tuple]({% slug howto_drill_down_navigation_startingfrom_root_tuple_pivotgrid %})
* [How to Expand Multiple Column Dimensions]({% slug howto_expand_multiple_column_dimensions_pivotgrid %})
* [How to Filter by Using the "include" Operator]({% slug howto_use_include_operator_pivotgrid %})
* [How to Make the Include fields Window Modal]({% slug howto_make_include_fields_window_modal_pivotgrid %})
* [How to Modify Measure Tag Captions]({% slug howto_modify_measure_tag_captions_pivotgrid %})
* [How to Reload PivotGrid Configuration Options]({% slug howto_reload_configuration_options_pivotgrid %})
* [How to Reset Expand State]({% slug howto_reset_expand_state_pivotgrid %})
* [How to Show Tooltip with Data Cell Information]({% slug howto_show_tooltip_withdata_cellinformation_pivotgrid %})
* [How to Translate PivotConfigurator Field Items]({% slug howto_translate_pivotconfigurator_messages_pivotgrid %})

For more runnable examples on the Kendo UI PivotGrid, browse its [**How To** documentation folder]({% slug howto_add_dimension_column_axis_pivotgrid %}).
