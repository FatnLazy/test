import requests
import json
import time
import copy
import sys
import os
import datetime
import logging
from program_aws import logs_program_aws

whole_seconds = 1


def send_logs(logs):
    if len(logs) == 0:
        return
    url = sys.argv[1]
    logztoken = sys.argv[2]
    logging.info(str(datetime.datetime.now()) + " sending " + str(len(logs)) + " logs size: " + str(sys.getsizeof(logs)))
    data_to_send = ""
    for msg in logs:
        data_to_send += "\n" + json.dumps(msg)
    try:
        result = requests.post(url, data=data_to_send, params={"token": logztoken})
        if result.status_code == 200:
            logging.info(str(datetime.datetime.now().time()) + " Successfully sent " + str(len(logs)) + " logs")
        else:
            logging.error(str(datetime.datetime.now().time()) + " error " + str(result.status_code) + ", couldn't send " + str(len(logs)) + " logs")
    except:
        logging.error(str(datetime.datetime.now().time()) + " error sending logs: " + str(logs))


def get_logs_for_program(prog):
    if prog['cross_fields']:
        temp_logs = [copy.deepcopy(prog['log_type'])]
        for field in prog['fields']:
            concat_logs = []
            for value in field['values']:
                for log in temp_logs:
                    new_log = copy.deepcopy(log)
                    write_to_nested_dict(new_log, field['field_name'], value)
                    concat_logs.append(new_log)
            temp_logs = concat_logs
    else:
        temp_logs = []
        values_count = min([len(field['values']) for field in prog['fields']])

        rolling_values = prog.get('rolling_values', values_count)
        if 'rolling_offset' in prog:
            prog['rolling_offset'] += rolling_values
        else:
            prog['rolling_offset'] = 0
        for i in range(rolling_values):
            new_log = copy.deepcopy(prog['log_type'])
            for field in prog['fields']:
                offset = (prog['rolling_offset'] + i) % len(field['values'])
                write_to_nested_dict(new_log, field['field_name'], field['values'][offset])
            temp_logs.append(new_log)
    return temp_logs

def get_logs_for_time():
    result_logs = []
    for prog in logs_program_aws:
        from_time = datetime.time(int(prog['from_time'].split(":")[0]), int(prog['from_time'].split(":")[1]), int(prog['from_time'].split(":")[2]))
        to_time = datetime.time(int(prog['to_time'].split(":")[0]), int(prog['to_time'].split(":")[1]), int(prog['to_time'].split(":")[2]))
        every = prog['every']

        simulate_time = datetime.datetime.now() + delta_time
        if to_time > from_time:
            generate = from_time <= simulate_time.time() < to_time
        else:
            generate = simulate_time.time() >= from_time or simulate_time.time() < to_time
        generate = generate and whole_seconds % every == 0
        if generate:
            result_logs += get_logs_for_program(prog)
    return result_logs


def write_to_nested_dict(dictionary, key, value):
    try:
        keys = key.split("|")
        for inner_key in range(len(keys) - 1):
            dictionary = dictionary[keys[inner_key]]
        dictionary[keys[len(keys) - 1]] = value
    except:
        logging.error("cant find key " + key + " in dictionary")


def read_from_nested_dict(dictionary, key):
    keys = key.split("|")
    for inner_key in range(len(keys) - 1):
        dictionary = dictionary[keys[inner_key]]
    return dictionary[keys[len(keys) - 1]]


if not os.path.exists('logger'):
    os.makedirs('logger')
delta_time = (datetime.datetime.strptime(sys.argv[3], '%H:%M:%S') if len(sys.argv) > 3 else datetime.datetime.now()) - datetime.datetime.now()

logging.basicConfig(filename='logger/events.log', level=logging.DEBUG)
logging.info("logs will be sent to account with token " + sys.argv[2] + " with timeshift of " + str(delta_time))
while True:
    logs_to_send = get_logs_for_time()
    print ("sening {} logs".format(len(logs_to_send)))
    send_logs(logs_to_send)
    for log in logs_to_send:
        logging.info(log)
    time.sleep(1)
    whole_seconds += 1
