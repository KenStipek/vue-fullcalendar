<template>
    <div ref="calendar" id="calendar"></div>
</template>

<script>
    require('fullcalendar')

    export default {
        props: {
            events: {
                default() {
                    return []
                },
            },

            eventSources: {
                default() {
                    return []
                },
            },

            editable: {
                default() {
                    return true
                },
            },

            selectable: {
                default() {
                    return true
                },
            },

            selectHelper: {
                default() {
                    return true
                },
            },

            header: {
                default() {
                    return {
                        left:   'prev,next today',
                        center: 'title',
                        right:  'month,agendaWeek,agendaDay'
                    }
                },
            },

            defaultView: {
                default() {
                    return 'agendaWeek'
                },
            },

            sync: {
                default() {
                    return false
                }
            },

            dayRender: {
                default() {
                    return (date, cell) => {};
                }
            },

            timezone: {
                default() {
                    return false;
                }
            },

            height: {
                default() {
                    return '';
                }
            },

            locale: {
                default() {
                    return 'en';
                }
            },
        },

        mounted() {
            const cal = $(this.$refs.calendar),
                self = this

            $(this.$refs.calendar).fullCalendar({
                header: this.header,
                defaultView: this.defaultView,
                editable: this.editable,
                selectable: this.selectable,
                selectHelper: this.selectHelper,
                aspectRatio: 2,
                height: this.height,
                timeFormat: 'hh:mmt',
                events: self.events,
                eventSources: self.eventSources,
                timezone: this.timezone,
                lang: this.locale,

                eventRender(event, element) {
                    if (this.sync) {
                        self.events = cal.fullCalendar('clientEvents')
                    }
                },

                eventDestroy(event) {
                    if (this.sync) {
                        self.events = cal.fullCalendar('clientEvents')
                    }
                },

                eventClick(event) {
                    self.$emit('event-selected', event)
                },

                eventDrop(event) {
                    self.$emit('event-drop', event)
                },

                eventResize(event) {
                    self.$emit('event-resize', event)
                },

                select(start, end, jsEvent) {
                    self.$emit('event-created', {
                        start,
                        end,
                        allDay: !start.hasTime() && !end.hasTime(),
                    })
                },

                dayClick(date, jsEvent, view) {
                    self.$emit('day-click', date, jsEvent, view);
                },

                dayRender(date, cell) {
                    self.dayRender(date, cell);
                },

                viewRender(view, element) {
                  self.$emit('view-render', view, element);
                },
            })
        },

        watch: {
            events: {
                deep: true,
                handler(val) {
                    this.$emit('reload-events');
                },
            },
            eventSources: {
                deep: true,
                handler(val) {
                    this.$emit('rebuild-sources');
                },
            },
        },

        methods: {
            gotoDate(date) {
              $(this.$refs.calendar).fullCalendar('gotoDate', date);
            },
        },

        created() {
            this.$on('remove-event', (event) => {
                $(this.$refs.calendar).fullCalendar('removeEvents', event.id)
            });

            this.$on('rerender-events', (event) => {
                $(this.$refs.calendar).fullCalendar('rerenderEvents')
            });

            this.$on('refetch-events', (event) => {
                $(this.$refs.calendar).fullCalendar('refetchEvents')
            });

            this.$on('render-event', (event) => {
                $(this.$refs.calendar).fullCalendar('renderEvent', event)
            });

            this.$on('reload-events', () => {
                $(this.$refs.calendar).fullCalendar('removeEvents')
                $(this.$refs.calendar).fullCalendar('addEventSource', this.events)
            });

            this.$on('rebuild-sources', () => {
                $(this.$refs.calendar).fullCalendar('removeEvents')
                this.eventSources.map(event => {
                    $(this.$refs.calendar).fullCalendar('addEventSource', event)
                })
            });
        },
    }
</script>
<style src="../../fullcalendar/dist/fullcalendar.css"></style>
