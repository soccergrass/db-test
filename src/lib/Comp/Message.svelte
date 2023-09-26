<script>
import { Row, Col, Collapse, Navbar, NavbarToggler, NavbarBrand, Nav, NavItem, Button, NavLink, Dropdown, DropdownToggle, DropdownMenu, DropdownItem, Form, InputGroup, Input } from 'sveltestrap'
import {push, link} from 'svelte-spa-router'
import {site, cert, user} from '../../dir/init.js'
// import {nanoid} from 'nanoid'
import MD5 from 'crypto-js/md5.js'
import { createEventDispatcher } from 'svelte'

const dispatch = createEventDispatcher()
export let auth
export let post
let text = ''
let err

async function handleSubmit(e){
    console.log(e)
    if(!text){
        return
    }
    let useText = text
    const stamp = Date.now()
    const sendMessage = {id: stamp + '-' + MD5(useText).toString(), replied: true, replies: false, stamp, iden: auth, file: '', text: useText}
    const makePost = site.get('posts').get(sendMessage.id).put(sendMessage, null, {opt: {cert}})
    const getPost = site.get('posts').get(post.id)
    makePost.get('to').get(post.id).put(getPost, null, {opt: {cert}})
    getPost.get('from').get(sendMessage.id).put(makePost, null, {opt: {cert}})
    if(!post.replies){
        getPost.get('replies').put(true, null, {opt: {cert}})
    }
    // site.get('categories').get(sendMessage.category).get(sendMessage.id).put(makePost, null, {opt: {cert}})
    // site.get('users').get(user.is.pub).get(sendMessage.id).put(sendMessage, null, {opt: {cert}})
    text = ''
    // dispatch('message', sendMessage)
    return sendMessage
}
</script>

{#if err}
    <Row>
        <Col>
            <p>{err}</p>
        </Col>
    </Row>
{/if}
<Row>
    <Col>
        <Form>
            <Input type="textarea" placeholder="post" id="text" bind:value={text}/>
            <Button type="button" on:click={(e) => {
                handleSubmit(e).then((data) => {dispatch('message', data)}).catch((e) => {console.error(e);err = e.message;setTimeout(() => {err = null}, 5000);})
            }}>submit</Button>
        </Form>
    </Col>
</Row>

<style></style>